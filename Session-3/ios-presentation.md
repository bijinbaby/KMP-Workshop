#Splash Screen

```
import SwiftUI
import shared

struct SplashScreen:View {
    
    @StateObject var viewModel = SplashScreen.ViewModel()
    
    let showNextScreen: () -> Void

    var body: some View{
        VStack{
            Image("news_logo", bundle: nil)
                .resizable()
                .aspectRatio(1.0, contentMode: .fit)
                .frame(width: 120)
        }.onChange(of: viewModel.state.isDataLoadingCompleted) { newValue in
            if(newValue){
                showNextScreen()
            }
        }
        .onAppear{
             viewModel.doAction(action: .LoadData)
        }
    }
}


```


# Home Page

```
import SwiftUI
import shared

struct HomeView: View {

    @State private var selectedTab: Tab = .article
    // Enumeration for tabs
       enum Tab {
           case article,bookmark
       }
    
    // Function to return the title based on the selected tab
       private func titleForTab(_ tab: Tab) -> String {
           switch tab {
           case .article:
               return "Articles"
           case .bookmark:
               return "Bookmarks"
           }
       }
    
    var body: some View {
        NavigationStack {
            TabView(selection:$selectedTab) {
                ArticleListView()
                    .tabItem {
                        Label("Articles", systemImage: "house.fill")
                    }
                    .tag(Tab.article)
                BookmarkListView()
                    .tabItem {
                        Label("Bookmarks", systemImage: "heart")
                    }
                    .tag(Tab.bookmark)
            }
            .navigationTitle(titleForTab(selectedTab))
            .navigationDestination(for: Article.self) { item in
                let viewModel = ArticleDetailView.ArticleDetailsViewModel(article: item)
                ArticleDetailView(viewModel: viewModel)
            }
        }
    }
}
```

# Articles List Page
## Articles List View
```
import SwiftUI

struct ArticleListView: View {
 
 @StateObject var viewModel = ArticleListViewModel()
 
 var body: some View {
     
     List(viewModel.state.news){ article in
         NavigationLink(value: article) {
             HStack{
                 AsyncImage(
                     url:URL(string:  article.urlToImage ?? "")
                 ){phase in
                     if let image = phase.image {
                         image.resizable()
                             .aspectRatio(contentMode: .fill)
                             .frame(width: 100, height: 100)
                             .clipped()
                             .clipShape(.capsule)
                     }
                 }
                 .frame(height: 100)
                 Text(article.title)
             }
         }
         .onAppear(){
             if article == viewModel.state.news.last{
                 viewModel.doAction(action: .fetchNews)
             }
         }
     }
     .onAppear(){
         viewModel.doAction(action: .fetchNews)
     }
     .listStyle(PlainListStyle())
 }
 }
```
## ArticleListViewModel
```
import Foundation
import shared

extension ArticleListView {
    struct State{
        var news = [Article]()
    }
    
    enum Action{
        case fetchNews
    }
    
    class ArticleListViewModel:ObservableObject{
       
        @Published var state: State = State()
        
        private var viewmodelHelper = ViemodelHelper.ArticleListViewModelHelper()
        private var isLoading: Bool = false
        private var pageIndex:Int32 = 1
        private var canLoadMorePages = true
        
        private var totalResults = 0
        private let pageSize = 100
        
        func doAction(action:Action){
            switch(action){
            case .fetchNews:
                fetchNews()
            }
        }
        
        private func fetchNews(){
            
            guard !self.isLoading && self.canLoadMorePages else { return }
            
            self.isLoading = true
            
            Task{
                let page = try? await viewmodelHelper.getNewsUsecase.invoke(sources: "bbc-news", page: pageIndex)
                if let articles = page?.articles{
                    self.totalResults += articles.count
                    self.canLoadMorePages = self.totalResults < page!.totalPages
                    self.pageIndex += 1
                    
                    await MainActor.run {
                        self.state.news.append(contentsOf: articles)
                        self.isLoading = false
                    }
                }
            }
        }
        
    }
}

extension Article: Identifiable{
    
}
```
# Detail Page
## ArtcleDetailView
```
import SwiftUI
import shared

struct ArticleDetailView: View {
    
    @StateObject var viewModel: ArticleDetailView.ViewModel
    
    var body: some View{
            ArticleDetailsViewContent(
                article: viewModel.article,
                isBookmarked: viewModel.state.isBookmarked,
                doAction: viewModel.doAction(action:)
            )
    }
}

struct ArticleDetailsViewContent: View{
    let article:Article
    let isBookmarked:Bool
    let doAction: (_ action:ArticleDetailView.Action)->Void
    
    var body: some View {
        VStack{
            AsyncImage(
                url:URL(string:  article.urlToImage ?? "")
            ){phase in
                if let image = phase.image {
                    image.resizable()
                        .aspectRatio(contentMode: .fill)
                        .frame(maxWidth: .infinity)
                        .clipped()
                }
            }
            .frame(maxWidth: .infinity)
            
            Text(article.title)
                .font(.largeTitle)
                .padding()
            Text(article.content ?? "")
                .font(.body)
                .padding()
            
        }
        .navigationBarTitleDisplayMode(.inline)
        .toolbar{
            ToolbarItem(placement: .navigationBarTrailing) {
                Button(action: {
                    doAction(.toggleBookmark)
                }) {
                    if isBookmarked{
                        Image(systemName: "heart.fill")
                    }else{
                        Image(systemName:  "heart")
                    }
                }
            }
            ToolbarItem(placement: .navigationBarTrailing) {
                Button(action: {
                    doAction(.shareArticle(link: article.url))
                }) {
                    Image(systemName: "person.2")
                }
            }
            ToolbarItem(placement: .navigationBarTrailing) {
                Button(action: {
                    doAction(.readFullArticle(link: ""))
                }) {
                    Image(systemName: "square.and.arrow.up") // Example icon:
                }
            }
        }
        .onAppear{
            doAction(.LoadData)
        }
    }
}
```
## ArticleDetailViewModel
```
import Foundation
import UIKit
import shared
import Combine

extension ArticleDetailView{

    struct State{
        var isBookmarked:Bool = false
    }

    enum Action{
        case LoadData
        case toggleBookmark
        case shareArticle(link:String)
        case readFullArticle(link:String)
    }

    class ViewModel: ObservableObject{

        private(set) var article:Article
        private var viewmodelHelper = ViemodelHelper.ArticleDetailsViewModelHelper()
        private var cancellables = Set<AnyCancellable>()
        
        @Published private(set) var state: State = State()
        
        init(article: Article) {
            self.article = article
        }

        func doAction(action:Action){
            switch(action){
            case .LoadData: loadData()
            case .toggleBookmark:
                toggleBookmark()
            case .shareArticle(link: let link):
                shareArticle(link: link)
            case .readFullArticle(link: let link):
                openInBrowser()
            }
        }

        private func loadData(){
            Task{
                do {
                    let value = try await viewmodelHelper.isBookmarkedUsecase.invoke(article: article)
                    await MainActor.run {
                        self.state.isBookmarked = value as! Bool
                    }
                } catch {
                }
            }
        }
        
        private func toggleBookmark() {
            state.isBookmarked = !state.isBookmarked
            Task{
                if (state.isBookmarked){
                    try await viewmodelHelper.addToBookmarksUsecase.invoke(article: article)
                }else{
                    try await viewmodelHelper.deleteFromBookmarksUsecase.invoke(article: article)
                }
            }
        }
        
        private func shareArticle(link: String){
            if let url = URL(string: link) {
                let activityViewController = UIActivityViewController(activityItems: [url],applicationActivities: nil)
                        
                // Get the current topmost view controller
                if let topController = UIApplication.shared.windows.first?.rootViewController {
                    topController.present(activityViewController, animated: true, completion: nil)
                }
            }
        }
        
        private func openInBrowser(){
            if let url = URL(string: article.url) {
                // Ensure the URL is valid and can be opened
                if UIApplication.shared.canOpenURL(url) {
                    UIApplication.shared.open(url, options: [:], completionHandler: nil)
                } else {
                    print("Invalid URL or cannot be opened")
                }
            }
        }
    }
}
```
# Bookmarks List Page

## BookmarksView
```
import Foundation
import SwiftUI
import shared

struct BookmarkListView: View {
 
 @StateObject var viewModel: BookmarkListViewModel = BookmarkListViewModel()
 
 var body: some View {
     List(viewModel.state.news){ article in
         NavigationLink(value: article) {
             HStack{
                 AsyncImage(
                     url:URL(string:  article.urlToImage ?? "")
                 ){phase in
                     if let image = phase.image {
                         image.resizable()
                             .aspectRatio(contentMode: .fill)
                             .frame(width: 100, height: 100)
                             .clipped()
                             .clipShape(.capsule)
                     }
                 }
                 .frame(height: 100)
                 Text(article.title)
             }
         }
     }
     .listStyle(PlainListStyle())
 }
}
```

## BookmarksViewModel
```
import Foundation
import shared
import Combine

extension BookmarkListView {
    
    struct State{
        var news = [Article]()
    }
    
    enum Action{
        case fetchBookmarks
    }
    
    class BookmarkListViewModel:ObservableObject{
                
        @Published var state: State = State()
        
        private var viewmodelHelper = ViemodelHelper.BookmarkListViewModelHelper()
        
        private var task: Task<(), Never>? = nil
        
        init(){
            doAction(action: .fetchBookmarks)
        }
        
        deinit{
            stopObserving()
        }
        
        func doAction(action: Action){
            switch action{
            case .fetchBookmarks:
                self.fetchBookMarkList()
            }
        }
        
        func fetchBookMarkList(){
            task = Task {
                for await articles in viewmodelHelper.getBookmarksUsecase.invoke(){
                    DispatchQueue.main.async {
                        self.state.news = articles
                    }
                }
            }
        }
        
        func stopObserving(){
            task?.cancel()
        }
        
    }
}
```
