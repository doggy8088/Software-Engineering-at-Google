
**CHAPTER 17**

# Code Search

# 第十七章 程式碼搜尋

**Written by Alexander Neubeck and Ben St. John**

**Edited by Lisa Carey**

Code Search is a tool for browsing and searching code at Google that consists of a frontend UI and various backend elements. Like many of the development tools at Google, it arose directly out of a need to scale to the size of the codebase. Code Search began as a combination of a grep-type tool[^1] for internal code with the ranking and UI of external Code Search[^2]. Its place as a key tool for Google developers was cemented by the integration of Kythe/Grok[^3], which added cross-references and the ability to jump to symbol definitions.

程式碼搜尋是用於在 Google 內部瀏覽和搜尋程式碼的工具，它由一個前端 UI 頁面和各種後端元件組成。就像Google的許多開發工具一樣，它直接源於程式碼庫規模的需求。程式碼搜尋開始是類似於 grep 型別工具的組合，用於帶有排行和 UI 的內部程式碼外部程式碼搜尋。透過 Kythe/Grok 的整合，它作為 Google 開發人員的關鍵工具的地位得到鞏固，他們增加了交叉引用和跳轉到符號定義的能力。

That integration changed its focus from searching to browsing code, and later development of Code Search was partly guided by a principle of 『answering the next question about code in a single click.』Now such questions as 『Where is this symbol defined?』, 『Where is it used?』, 『How do I include it?』, 『When was it added to the codebase?』, and even ones like 『Fleet-wide, how many CPU cycles does it consume?』 are all answerable with one or two clicks.

這種整合將重點從搜尋轉移到瀏覽程式碼，後來程式碼搜尋的發展部分遵循『單擊回答下一個關於程式碼的問題』的原則。現在諸如『這個符號在哪裡定義？』，『它在哪裡使用？』、『我如何包含它？』、『它是什麼時候新增到程式碼庫中的？』，甚至像『Fleet-wide，它消耗多少 CPU 週期？』之類的問題。只需單擊一兩次即可得到答案。

In contrast to integrated development environments (IDEs) or code editors, Code Search is optimized for the use case of reading, understanding, and exploring code at scale. To do so, it relies heavily on cloud based backends for searching content and resolving cross-references. 

與整合開發環境 (IDE) 或程式碼編輯器相比，程式碼搜尋針對大規模閱讀、理解和探索程式碼的用例進行了最佳化。為此，它嚴重依賴基於雲的後端來搜尋內容和解決交叉引用。

In this chapter, we」ll look at Code Search in more detail, including how Googlers use it as part of their developer workflows, why we chose to develop a separate web tool for code searching, and examine how it addresses the challenges of searching and browsing code at Google repository scale. 

在本章中，我們將更詳細地瞭解程式碼搜尋，包括 Google 員工如何將其作為開發人員工作流程的一部分，為什麼我們選擇開發一個單獨的網路工具來進行程式碼搜尋，並研究它如何在 Google 儲存庫規模下解決搜尋和瀏覽程式碼問題。

> [^1]: GSearch originally ran on Jeff Dean」s personal computer, which once caused company-wide distress when he went on vacation and it was shut down!
>
> 1 GSearch最初在Jeff Dean的個人電腦上執行，當他去度假時，曾經引起全公司的困擾。他去度假時，這台電腦就被關閉了！這曾經造成了整個公司的困擾。
>
> [^2]: Shut down in 2013; see `https://en.wikipedia.org/wiki/Google_Code_Search`.
>
> 2 在2013年關閉；見`https://en.wikipedia.org/wiki/Google_Code_Search`。
>
> [^3]: Now known as Kythe, a service that provides cross-references (among other things): the uses of a particular code symbol—for example, a function—using the full build information to disambiguate it from other ones with the same name.
>
> 3 現在被稱為Kythe，一個提供交叉引用的服務（除其他外）：一個特定的程式碼符號的用途--例如，一個函式--使用完整的建立訊息，將其與其他同名的符號區分開來。

## The Code Search UI 程式碼搜尋使用者介面

The search box is a central element of the Code Search UI (see Figure 17-1), and like web search, it has 『suggestions』 that developers can use for quick navigation to files, symbols, or directories. For more complex use cases, a results page with code snippets is returned. The search itself can be thought of as an instant 『find in files』 (like the Unix grep command) with relevance ranking and some code-specific enhancements like proper syntax highlighting, scope awareness, and awareness of comments and string literals. Search is also available from the command line and can be incorporated into other tools via a Remote Procedure Call (RPC) API. This comes in handy when post-processing is required or if the result set is too large for manual inspection.

搜尋框是程式碼搜尋 UI 的中心元素（見圖 17-1），與 Web 搜尋一樣，它有『建議』，開發人員可以使用這些『建議』快速導向到檔案、符號或目錄。對於更復雜的用例，將回傳帶有程式碼片段的結果頁面。搜尋本身可以被認為是即時的『在檔案中查詢』（如 Unix grep 指令），具有相關性排行和一些特定於程式碼的增強功能，如正確的語法突出顯示、範圍感知以及註解和字串文字的感知。搜尋也可以在指令行使用，並且可以透過遠端過程呼叫 (RPC) API 併入其他工具。當需要事後處理或結果集太大而無法手動檢查時，這會派上用場。

![Figure 17-1](./images/Figure%2017-1.png)

When viewing a single file, most tokens are clickable to let the user quickly navigate to related information. For example, a function call will link to its function definition, an imported filename to the actual source file, or a bug ID in a comment to the corresponding bug report. This is powered by compiler-based indexing tools like Kythe. Clicking the symbol name opens a panel with all the places the symbol is used. Similarly, hovering over local variables in a function will highlight all occurrences of that variable in the implementation. 

檢視單個檔案時，大多數標記都是可單擊的，以便使用者快速導向到相關訊息。例如，函式呼叫將連結到其函式定義、匯入的檔案名到實際源檔案，或相應錯誤報告註解中的錯誤 ID。這由 Kythe 等基於編譯器的索引工具提供支援。單擊符號名稱會開啟一個面板，其中包含使用該符號的所有位置。同樣，將滑鼠懸停在函式中的區域性變數上將突出顯示該變數在實現中的所有出現。

Code Search also shows the history of a file, via its integration with Piper (see Chapter 16). This means seeing older versions of the file, which changes have affected it,  who wrote them, jumping to them in Critique (see Chapter 19), diffing versions of files, and the classic 『blame』 view if desired. Even deleted files can be seen from a directory view. 

程式碼搜尋還可以顯示檔案的歷史記錄，透過與 Piper 的整合（參見第 16 章）。這意味著檢視檔案的舊版本，哪些更改影響了它，誰編寫了它們，在 Critique 中跳轉到它們（參見第 19 章），區分檔案的版本，以及經典的『blame』檢視（如果需要）。甚至可以從目錄檢視中看到已刪除的檔案。

## How Do Googlers Use Code Search? Google 員工如何使用程式碼搜尋？

Although similar functionality is available in other tools, Googlers still make heavy use of the Code Search UI for searching and file viewing and ultimately for understanding code.[^4] The tasks engineers try to complete with Code Search can be thought of answering questions about code, and recurring intents become visible.[^5]

儘管其他工具中也有類似的功能，但 Google 員工仍然大量使用程式碼搜尋 UI 進行搜尋和檔案檢視，並最終用於理解程式碼。工程師嘗試使用程式碼搜尋完成任務被認為是回答有關程式碼的問題，以及重複的意圖變得可見。

> [^4]: There is an interesting virtuous cycle that a ubiquitous code browser encourages: writing code that is easy to browse. This can mean things like not nesting hierarchies too deep, which requires many clicks to move from call sites to actual implementation, and using named types rather than generic things like strings or integers, because it」s then easy to find all usages.
>
> 4 無處不在的程式碼瀏覽器鼓勵一個有趣的良性迴圈：編寫易於瀏覽的程式碼。這可能意味著不要把層次巢狀得太深，因為這需要多次點選才能從呼叫站點轉移到實際的實現；使用命名的型別而不是像字串或整數這樣的通用類型，因為這樣就很容易找到所有的用法。
>
> [^5]: Sadowski, Caitlin, Kathryn T. Stolee, and Sebastian Elbaum. 『How Developers Search for Code: A Case Study』 In Proceedings of the 2015 10th Joint Meeting on Foundations of Software Engineering (ESEC/FSE 2015). `https://doi.org/10.1145/2786805.2786855`.
>
> 5 Sadowski, Caitlin, Kathryn T. Stolee, and Sebastian Elbaum. "開發者如何搜尋程式碼。A Case Study" In Proceedings of the 2015 10th Joint Meeting on Foundations of Software Engineering (ESEC/FSE 2015). `https://doi.org/10.1145/2786805.2786855`.

### Where?

### 哪裡？

About 16% of Code Searches try to answer the question of where a specific piece of information exists in the codebase; for example, a function definition or configuration, all usages of an API, or just where a specific file is in the repository. These questions are very targeted and can be very precisely answered with either search queries or by following semantic links, like 『jump to symbol definition.』 Such questions often arise during larger tasks like refactorings/cleanups or when collaborating with other engineers on a project. Therefore, it is essential that these small knowledge gaps are addressed efficiently.  

大約 16% 的程式碼搜尋試圖解答特定訊息在程式碼庫中的位置的問題；例如，函式定義或設定、API 的所有用法，或者特定檔案在儲存庫中的位置。這些問題非常有針對性，可以透過搜尋查詢或遵循語義連結（例如『跳轉到符號定義』）來非常精確地回答。此類問題經常出現在重構/清理等大型任務中，或者在與其他工程師合作進行專案時。因此，有效解決這些小的知識差距至關重要。

Code Search provides two ways of helping: ranking the results, and a rich query language. Ranking addresses the common cases, and searches can be made very specific (e.g., restricting code paths, excluding languages, only considering functions) to deal with rarer cases. 

程式碼搜尋提供了兩種幫助方式：對結果進行排行，以及豐富的查詢語言。排行解決了常見問題，並且可以進行非常具體的搜尋（例如，限制程式碼路徑，排除語言，僅考慮功能）以處理罕見情況。

The UI makes it easy to share a Code Search result with colleagues. So, for code reviews, you can simply include the link—for example, 『Have you considered using this specialized hash map: cool_hash.h? This is also very useful for documentation, in bug reports, and in postmortems and is the canonical way of referring to code within Google. Even older versions of the code can be referenced, so links can stay valid as the codebase evolves. 

使用者介面讓同事之間共享程式碼搜尋結果變得容易。因此，對於程式碼審查，你可以簡單地包含連結——例如，『你是否考慮過使用這個專門的雜湊對映：cool_hash.h？這對於文件、錯誤報告和事後分析也非常有用，並且是在 Google 中引用程式碼的規範方式。甚至可以引用舊版本的程式碼，因此連結可以隨著程式碼庫的發展而保持有效。

### What? 什麼？

Roughly one quarter of Code Searches are classic file browsing, to answer the question of what a specific part of the codebase is doing. These kinds of tasks are usually more exploratory, rather than locating a specific result. This is using Code Search to read the source, to better understand code before making a change, or to be able to understand someone else」s change.  

大約四分之一的程式碼搜尋是典型的檔案瀏覽，回答程式碼特定部分在做什麼的問題。這些型別的任務通常更具探索性，而不是定位特定的結果。使用程式碼搜尋來閱讀原始碼，以便在進行更改之前更好地理解程式碼，或者能夠理解其他人的更改。

To ease these kinds of tasks, Code Search introduced browsing via call hierarchies and quick navigation between related files (e.g., between header, implementation, test, and build files). This is about understanding code by easily answering each of the many questions a developer has when looking at it. 

為了簡化這些型別的任務，程式碼搜尋引入了呼叫層次結構瀏覽和相關檔案之間的快速導向（例如，在標題、實現、測試和建立檔案之間）。透過回答開發人員在檢視程式碼時遇到的許多問題來理解程式碼。

### How? 如何做？

The most frequent use case—about one third of Code Searches—are about seeing examples of how others have done something. Typically, a developer has already found a specific API (e.g., how to read a file from remote storage) and wants to see how the API should be applied to a particular problem (e.g., how to set up the remote connection robustly and handle certain types of errors). Code Search is also used to find the proper library for specific problems in the first place (e.g., how to compute a fingerprint for integer values efficiently) and then pick the most appropriate implementation. For these kinds of tasks, a combination of searches and cross-reference browsing are typical. 

最常見的用例—大約三分之一的程式碼搜尋是關於檢視其他人如何做某事的範例。通常，開發人員已經找到了特定的 API（例如，如何從遠端儲存中讀取檔案）並希望瞭解如何將 API 應用於特定問題（例如，如何穩健地建立遠端連線並處理某些問題）。程式碼搜尋還用於首先為特定問題找到合適的函式庫（例如，如何有效地計算整數值的指紋），然後選擇最合適的實現。對於這些型別的任務，搜尋和交叉引用瀏覽的組合是典型的。

### Why？為什麼？

Related to what code is doing, there are more targeted queries around why code is behaving differently than expected. About 16% of Code Searches try to answer the question of why a certain piece of code was added, or why it behaves in a certain way. Such questions often arise during debugging; for example, why does an error occur under these particular circumstances? 

與程式碼在做什麼有關，關於為什麼程式碼的行為與預期不同，有更多有針對性的查詢。大約 16% 的程式碼搜尋試圖回答為什麼要新增某段程式碼，或者為什麼它以某種方式執行的問題。除錯過程中經常會出現這樣的問題；例如，為什麼在這些特定情況下會發生錯誤？

An important capability here is being able to search and explore the exact state of the codebase at a particular point in time. When debugging a production issue, this can mean working with a state of the codebase that is weeks or months old, while debugging test failures for new code usually means working with changes that are only minutes old. Both are possible with Code Search. 

這裡的一個重要功能是能夠在特定時間點搜尋和探索程式碼庫的確切狀態。在除錯生產問題時，這可能意味著使用幾周或幾個月前的程式碼庫狀態，而除錯新程式碼的測試失敗通常意味著使用僅幾分鐘前的更改。兩者都可以透過程式碼搜尋實現。

### Who and When? 誰？什麼時候？

About 8% of Code Searches try to answer questions around who or when someone introduced a certain piece of code, interacting with the version control system. For example, it」s possible to see when a particular line was introduced (like Git」s 『blame』) and jump to the relevant code review. This history panel can also be very useful in finding the best person to ask about the code, or to review a change to it.[^6]

大約 8% 的程式碼搜尋試圖回答有關誰或何時引入某段程式碼的問題，並與版本控制系統進行互動。例如，可以檢視何時引入了特定行（如 Git 的『blame』）並跳轉到相關的程式碼審查。這個歷史面板對於尋找最好的人來詢問程式碼或審查對它的更改也非常有用。

> [^6]: That said, given the rate of commits for machine-generated changes, naive 『blame』 tracking has less value than it does in more change-averse ecosystems.
>
> 6也就是說，考慮到機器生成的更改的送出率，天真的『指責』跟蹤比在更厭惡更改的生態系統中的價值要小。

## Why a Separate Web Tool? 為什麼要使用單獨的 Web 工具？

Outside Google, most of the aforementioned investigations are done within a local IDE. So, why yet another tool? 

在 Google 之外，上述大部分實現都是在本地IDE。那麼，為什麼還需要有另一個工具呢？

### Scale 規模

The first answer is that the Google codebase is so large that a local copy of the full codebase—a prerequisite for most IDEs—simply doesn」t fit on a single machine. Even before this fundamental barrier is hit, there is a cost to building local search and cross-reference indices for each developer, a cost often paid at IDE startup, slowing developer velocity. Or, without an index, one-off searches (e.g., with grep) can become painfully slow. A centralized search index means doing this work once,  upfront, and means investments in the process benefit everyone. For example, the Code Search index is incrementally updated with every submitted change, enabling index construction with linear cost.[^7]

第一個答案是 Google 程式碼庫規模太大，以至於完整程式碼庫的本地副本（大多數 IDE 的先決條件）根本不適合單台機器。即使在這個基本障礙之前，為每個開發人員建立本地搜尋和交叉引用索引也是有成本的，這通常在 IDE 啟動時降低了開發人員的效率。如果沒有索引，一次性搜尋（例如，使用 grep）可能會變得非常緩慢。集中式搜尋索引意味著一次性完成這項工作，並且意味著對流程的投資使每個人都受益。例如，程式碼搜尋索引會隨著每次送出的更改而增量更新，從而能夠以線性成本建立索引。

In normal web search, fast-changing current events are mixed with more slowly changing items, such as stable Wikipedia pages. The same technique can be extended to searching code, making indexing incremental, which reduces its cost and allows changes to the codebase to be visible to everyone instantly. When a code change is submitted, only the actual files touched need to be reindexed, which allows parallel and independent updates to the global index. 

在正常的網路搜尋中，快速變化的當前事件與變化較慢的專案混合在一起，例如穩定的維基百科頁面。同樣的技術可以擴充套件到搜尋程式碼，使索引增加，從而降低成本，並允許對程式碼庫的更改立即對所有人可見。送出程式碼更改時，只需要對實際觸及的檔案進行重新索引，這允許對全域索引進行並行和獨立的更新。

Unfortunately, the cross-reference index cannot be instantly updated in the same way. Incrementality isn」t possible for it, as any code change can potentially influence the entire codebase, and in practice often does affect thousands of files. Many (nearly all of Google」s) full binaries need to be built[^8] (or at least analyzed) to determine the full semantic structure. It uses a ton of compute resources to produce the index daily (the current frequency). The discrepancy between the instant search index and the daily  cross-reference index is a source of rare but recurring issues for users.

不幸的是，交叉引用索引不能以相同的方式立即更新。增量是不可能的，因為任何程式碼更改都可能影響整個程式碼庫，實際上經常會影響數千個檔案。需要建立（或至少分析）許多（幾乎所有 Google 的）完整二進位檔案以確定完整的語義結構。它每天使用大量計算資源（當前頻率）生成索引。即時搜尋索引和每日交叉引用索引之間的差異是使用者罕見但反覆出現的問題的根源。

> [^7]: For comparison, the model of 『every developer has their own IDE on their own workspace do the indexing calculation』 scales roughly quadratically: developers produce a roughly constant amount of code per unit time, so the codebase scales linearly (even with a fixed number of developers). A linear number of IDEs do linearly more work each time—this is not a recipe for good scaling.
>
> 7  相比之下，『每個開發人員在自己的工作空間中都有自己的IDE，並進行索引計算』的模型大致按二次方進行擴充套件：開發人員每單位時間生成的程式碼量大致恆定，因此程式碼庫可以線性擴充套件（即使有固定數量的開發人員）。線性數量的IDE每次都會做線性更多的工作，但這並不是實現良好擴充套件的秘訣。
>
> [^8]: Kythe instruments the build workflow to extract semantic nodes and edges from source code. This extraction process collects partial cross-reference graphs for each individual build rule. In a subsequent phase, these partial graphs are merged into one global graph and its representation is optimized for the most common queries (go-to-definition, find all usages, fetch all decorations for a file). Each phase—extraction and post processing—is roughly as expensive as a full build; for example, in case of Chromium, the construction of the Kythe index is done in about six hours in a distributed setup and therefore too costly to be constructed by every developer on their own workstation. This computational cost is the why the Kythe index is computed only once per day.
>
> 8 Kyth使用建立工作流從原始碼中提取語義節點和邊緣。這個提取過程為每個單獨的生成規則收集部分交叉引用圖。在隨後的階段中，這些區域性圖合併為一個全域圖，並針對最常見的查詢對其表示進行最佳化（轉到定義，查詢所有用法，獲取檔案的所有修飾）。每個階段的提取和後處理成本大致與完整建立一樣高；例如，對於Chromium，Kythe索引的建立在分散式設定中大約需要六個小時，因此每個開發人員都無法在自己的工作站上建立，成本太高。這就是為什麼Kythe指數每天只計算一次的原因。

### Zero Setup Global Code View  零設定全域程式碼檢視

Being able to instantly and effectively browse the entire codebase means that it」s very easy to find relevant libraries to reuse and good examples to copy. For IDEs that construct indices at startup, there is a pressure to have a small project or visible scope to reduce this time and avoid flooding tools like autocomplete with noise. With the Code Search web UI, there is no setup required (e.g., project descriptions, build environment), so it」s also very easy and fast to learn about code, wherever it occurs, which improves developer efficiency. There」s also no danger of missing code dependencies;  for example, when updating an API, reducing merge and library versioning issues. 

能夠立即有效地瀏覽整個程式碼庫意味著很容易找到相關的函式庫來重用和好的例子來復制。對於在啟動時建立索引的 IDE，有一個挑戰是，要有一個小專案或可見範圍來減少啟動時間，並避免像自動完成這樣的工具氾濫而產生噪音。使用程式碼搜尋 Web UI，無需設定（例如，專案描述、建立環境），因此無論程式碼出現在何處，都可以非常輕鬆快速地瞭解程式碼，從而提高開發人員效率。也沒有丟失程式碼依賴的危險；例如，在更新 API 時，減少合併和庫版本控制問題。

### Specialization 專業化

Perhaps surprisingly, one advantage of Code Search is that it is not an IDE. This means that the user experience (UX) can be optimized for browsing and understanding code, rather than editing it, which is usually the bulk of an IDE (e.g., keyboard shortcuts, menus, mouse clicks, and even screen space). For example, because there isn」t an editor」s text cursor, every mouse click on a symbol can be made meaningful(e.g., show all usages or jump to definition), rather than as a way to move the cursor. This advantage is so large that it」s extremely common for developers to have multiple Code Search tabs open at the same time as their editor.

也許令人驚訝的是，程式碼搜尋的一個優點是它不是 IDE。這意味著使用者體驗 (UX) 可以針對瀏覽和理解程式碼進行最佳化，而不是像 IDE 的大部分內容那樣編輯它（例如，鍵盤快捷鍵、選單、滑鼠點選，甚至螢幕空間）。例如，由於沒有編輯器的文字游標，每次滑鼠單擊符號都可以變得有意義（例如，顯示所有用法或跳轉到定義），而不是作為移動游標的一種方式。這個優勢是如此之大，以至於開發人員在使用編輯器的同時開啟多個程式碼搜尋頁籤是非常常見的。

### Integration with Other Developer Tools 與其他開發者工具整合

Because it is the primary way to view source code, Code Search is the logical platform for exposing information about source code. It frees up tool creators from needing to create a UI for their results and ensures the entire developer audience will know of their work without needing to advertise it. Many analyses run regularly over the entire Google codebase, and their results are usually surfaced in Code Search. For example, for many languages, we can detect 『dead』 (uncalled) code and mark it as such when the file is browsed.

因為它是檢視原始碼的主要方式，所以程式碼搜尋是公開原始碼訊息的邏輯平台。它使工具建立者無需為其結果建立 UI，並確保整個開發人員無需宣傳即可瞭解他們的工作。許多分析會定期在整個 Google 程式碼庫中執行，它們的結果通常會出現在程式碼搜尋中。例如，對於許多語言，我們可以檢測『死』（未呼叫）程式碼，並在瀏覽檔案時將其標記為死程式碼。

In the other direction, the Code Search link to a source file is considered its canonical 『location.』 This is useful for many developer tools (see Figure 17-2). For example, log file lines typically contain the filename and line number of the logging statement. The production log viewer uses a Code Search link to connect the log statement back to the producing code. Depending on the available information, this can be a direct link to a file at a specific revision, or a basic filename search with the corresponding line number. If there is only one matching file, it is opened at the corresponding line number. Otherwise, snippets of the desired line in each of the matching files are rendered.

另一方面，指向源檔案的程式碼搜尋連結被認為是其規範的『位置』。這對許多開發工具很有用（見圖 17-2）。例如，日誌檔案行通常包含日誌記錄語句的檔案名和行號。生產日誌檢視器使用程式碼搜尋連結將日誌連線回生產程式碼。根據可用訊息，這可以是指向特定修訂檔案的直接連結，也可以是具有相應行號的基本檔案名搜尋。如果只有一個對應檔案，則在相應的行號處開啟。否則，將呈現每個對應檔案中所需行的片段。

![Figure 17-2](./images/Figure%2017-2.png)

Similarly, stack frames are linked back to source code whether they are shown within a crash reporting tool or in log output, as shown in Figure 17-3. Depending on the programming language, the link will utilize a filename or symbol search. Because the snapshot of the repository at which the crashing binary was built is known, the search can actually be restricted to exactly this version. That way, links remain valid for a long time period, even if the corresponding code is later refactored or deleted.

類似地，堆疊幀被連結回原始碼，無論它們是顯示在崩潰報告工具中還是顯示在日誌輸出中，如圖 17-3 所示。根據程式語言，連結將使用檔案名或符號搜尋。因為建立崩潰二進位檔案的儲存庫的快照是已知的，所以實際上可以將搜尋限制在這個版本。這樣，即使相應的程式碼後來被重構或刪除，連結也會在很長一段時間內保持有效。

![Figure 17-3](./images/Figure%2017-3.png)

Compilation errors and tests also typically refer back to a code location (e.g., test X in file at line). These can be linkified even for unsubmitted code given that most development happens in specific cloudvisible workspaces that are accessible and searchable by Code Search.

編譯錯誤和測試通常還參考程式碼位置（例如，測試 X在檔案中行號）。即使對於未送出的程式碼，這些也可以連結起來，因為大多數開發都發生在特定的雲可見工作區中，這些工作區可以透過程式碼搜尋訪問和搜尋。

Finally, codelabs and other documentation refer to APIs, examples, and implementations. Such links can be search queries referencing a specific class or function, which remain valid when the file structure changes. For code snippets, the most recent implementation at head can easily be embedded into a documentation page, as demonstrated in Figure 17-4, without the need to pollute the source file with additional documentation markers.

最後，程式碼實驗室和其他文件是指 API、範例和實現。此類連結可以是引用特定類別或函式的搜尋查詢，當檔案結構更改時它們仍然有效。對於程式碼片段，最新的實現可以很容易地嵌入到文件頁面中，如圖 17-4 所示，而無需使用額外的文件標記汙染源檔案。

![Figure 17-4](./images/Figure%2017-4.png)

### API Exposure API 暴露

Code Search exposes its search, cross-reference, and syntax highlighting APIs to tools, so tool developers can bring those capabilities into their tools without needing to reimplement them. Further, plug-ins have been written to provide search and cross-references to editors and IDEs such as vim, emacs, and IntelliJ. These plugins restore some of the power lost due to being unable to locally index the codebase, and give back some developer productivity.

程式碼搜尋將其搜尋、交叉引用和語法高亮 API 公開給工具，因此工具開發人員可以將這些功能帶入他們的工具中，而無需重新實現它們。此外，還編寫了外掛來提供對編輯器和 IDE（例如 vim、emacs 和 IntelliJ）的搜尋和交叉引用。這些外掛恢復了由於無法在本地索引程式碼庫而損失的一些效率，並提升了一些開發人員的生產力。

## Impact of Scale on Design 規模對設計的影響

In the previous section, we looked at various aspects of the Code Search UI and why it」s worthwhile having a separate tool for browsing code. In the following sections, we look a bit behind the scenes of the implementation. We first discuss the primary challenge—scaling—and then some of the ways the large scale complicates making a good product for searching and browsing code. After that, we detail how we addressed some of those challenges, and what trade-offs were made when building Code Search.

在上一節中，我們研究了程式碼搜尋 UI 的各個方面，以及為什麼需要擁有一個單獨的工具來瀏覽程式碼。在接下來的部分中，我們會稍微瞭解一下程式碼搜尋實現的幕後情況。我們首先討論了主要挑戰——擴充套件——然後討論了幾種大規模複雜化建立搜尋和瀏覽程式碼好產品的方式。之後，我們詳細介紹了我們如何應對其中的一些挑戰，以及在建立程式碼搜尋時做出了哪些權衡。

The biggest[^9] scaling challenge for searching code is the corpus size. For a small repository of a couple megabytes, a brute-force search with grep search will do. When hundreds of megabytes need to be searched, a simple local index can speed up search by an order of magnitude or more. When gigabytes or terabytes of source code need to be searched, a cloud-hosted solution with multiple machines can keep search times reasonable. The utility of a central solution increases with the number of developers using it and the size of the code space.

搜尋程式碼的最大挑戰是語料庫大小。對於幾MB的小型儲存庫，使用 grep 搜尋的蠻力搜尋就可以了。當需要搜尋數百MB時，一個簡單的本地索引可以將搜尋速度提高一個數量級或更多。當需要搜尋千MB或千MB的原始碼時，具有多台機器的雲託管解決方案可以使搜尋時間保持合理。中央解決方案的實用性隨著使用它的開發人員的數量和程式碼空間的大小而增加。

> [^9]: Because queries are independent, more users can be addressed by having more servers.
>
> 9 因為查詢是獨立的，所以可以透過擁有更多的伺服器來解決更多的使用者。

### Search Query Latency 搜尋查詢延遲

Although we take as a given that a fast and responsive UI is better for the user, low latency doesn」t come for free. To justify the effort, one can weigh it against the saved engineering time across all users. Within Google, we process much more than one million search queries from developers within Code Search per day. For one million queries, an increase of just one second per search request corresponds to about 35 idle full-time engineers every day. In contrast, the search backend can be built and maintained with roughly a tenth of these engineers. This means that with about 100,000 queries per day (corresponding to less than 5,000 developers), just the one-second latency argument is something of a break-even point.

儘管我們認為快速回應的 UI 對使用者來說更好，但低延遲並不是免費的。為了證明這一努力的合理性，可以將其與所有使用者節省的工程時間進行權衡。在 Google 內部，我們每天在程式碼搜尋中處理超過一百萬個來自開發人員的搜尋查詢。對於一百萬個查詢，每個搜尋請求僅增加一秒，就相當於每天大約有 35 名空閒的全職工程師。相比之下，搜尋後端可以由大約十分之一的工程師來建立和維護。這意味著每天大約有 100,000 次查詢（對應於不到 5,000 名開發人員），僅一秒鐘的延遲引數就可以達到收支平衡點。

In reality, the productivity loss doesn」t simply increase linearly with latency. A UI is considered responsive if latencies are below 200 ms. But after just one second, the developer」s attention often begins to drift. If another 10 seconds pass, the developer is likely to switch context completely, which is generally recognized to have high productivity costs. The best way to keep a developer in the productive 『flow』 state is by targeting sub–200 ms end-to-end latency for all frequent operations and investing in the corresponding backends.

實際上，生產力損失並不僅僅隨著延遲線性增加。如果延遲低於 200 毫秒，則認為 UI 是回應式的。但僅僅一秒鐘後，開發人員的注意力往往開始轉移。如果再過 10 秒，開發人員很可能會完全切換上下文，這通常被認為具有很高的生產力成本。讓開發人員保持高效『流動』狀態的最佳方法是將所有頻繁操作的端到端延遲設定在 200 毫秒以下，並投資於相應的後端。

A large number of Code Search queries are performed in order to navigate the codebase. Ideally, the 『next』 file is only a click away (e.g., for included files, or symbol definitions), but for general navigation, instead of using the classical file tree, it can be much faster to simply search for the desired file or symbol, ideally without needing to fully specify it, and suggestions are provided for partial text. This becomes increasingly true as the codebase (and file tree) grows.

執行大量程式碼搜尋查詢來導向程式碼庫。理想情況下，『下一個』檔案只需單擊一下即可（例如，對於包含的檔案或符號定義），但對於一般導向，不需要使用經典檔案樹，簡單地搜尋所需的檔案或符號會快得多，理想情況下不需要完全指定它，會為部分文字提供聯想查詢。隨著程式碼庫（和檔案樹）的增長，這變得越來越正確。

Normal navigation to a specific file in another folder or project requires several user interactions. With search, just a couple of keystrokes can be sufficient to get to the relevant file. To make search this effective, additional information about the search context (e.g., the currently viewed file) can be provided to the search backend. The context can restrict the search to files of a specific project, or influence ranking by preferring files that are in proximity to other files or directories. In the Code Search UI,[^10] the user can predefine multiple contexts and quickly switch between them as needed. In editors, the open or edited files are implicitly used as context to prioritize search results in their proximity.

正常導向到另一個資料夾或專案中的特定檔案需要多次使用者互動。使用搜尋，只需幾次點選即可訪問相關檔案。為了使搜尋有效，可以將有關搜尋上下文的附加訊息（例如，當前檢視的檔案）提供給搜尋後端。上下文可以將搜尋限制為特定專案的檔案，或者透過優先選擇靠近其他檔案或目錄的檔案來影響排行。在程式碼搜尋 UI 中， 使用者可以預定義多個上下文並根據需要在它們之間快速切換。在編輯器中，開啟或編輯的檔案被隱式用作上下文，以優先考慮搜尋結果的接近程度。

One could consider the power of the search query language (e.g., specifying files,using regular expressions) as another criteria; we discuss this in the trade-offs section a little later in the chapter.

可以將搜尋查詢語言的功能（例如，指定檔案、使用正則表示式）視為另一個標準；我們將在本章稍後的權衡部分討論這個問題。

> [^10]: The Code Search UI does also have a classical file tree, so navigating this way is also possible.
>
> 10 程式碼搜尋使用者介面也有一個經典的檔案樹，所以用這種方式導向也是可以的。

### Index Latency 索引延遲

Most of the time, developers won」t notice when indices are out of date. They only care about a small subset of code, and even for that they generally won」t know whether there is more recent code. However, for the cases in which they wrote or reviewed the corresponding change, being out of sync can cause a lot of confusion. It tends not to matter whether the change was a small fix, a refactoring, or a completely new piece of code—developers simply expect a consistent view, such as they experience in their IDE for a small project.

大多數時候，開發人員不會注意到索引何時過期。他們只關心一小部分程式碼，即便如此，他們通常也不知道是否有更新的程式碼。但是，對於他們編寫或審查相應更改的情況，不同步可能會導致很多混亂。更改是小修復、重構還是全新的程式碼片段往往並不重要——開發人員只期望一個一致的檢視，例如他們在 IDE 中為一個小專案所體驗的。

When writing code, instant indexing of modified code is expected. When new files, functions, or classes are added, not being able to find them is frustrating and breaks the normal workflow for developers used to perfect cross-referencing. Another example are search-and-replace–based refactorings. It is not only more convenient when the removed code immediately disappears from the search results, but it is also essential that subsequent refactorings take the new state into account.  When working with a centralized VCS, a developer might need instant indexing for submitted code if the previous change is no longer part of the locally modified file set.

編寫程式碼時，需要對修改後的程式碼進行即時索引。當新增新檔案、函式或類時，找不到它們是令人沮喪的，並且破壞了用於完善交叉引用的開發人員的正常工作流程。另一個例子是基於搜尋和替換的重構。刪除的程式碼立即從搜尋結果中消失不僅更方便，而且後續重構考慮新狀態也很重要。使用集中式 VCS 時，如果先前的更改不再是本地修改檔案集的一部分，則開發人員可能需要對送出的程式碼進行即時索引。

Conversely, sometimes it」s useful to be able to go back in time to a previous snapshot of the code; in other words, a release. During an incident, a discrepancy between the index and the running code can be especially problematic because it can hide real causes or introduce irrelevant distractions. This is a problem for cross-references because the current technology for building an index at Google」s scale simply takes hours, and the complexity means that only one 『version』 of the index is kept. Although some patching can be done to align new code with an old index, this is still an issue to be solved.

相反，有時能夠及時回到之前的程式碼快照很有用；換句話說，在事件期間釋放，索引和執行程式碼之間的差異可能會是問題，因為它可以隱藏真正的原因或引入不相關的干擾。這對於交叉引用來說是一個問題，因為目前在 Google 規模上建立索引的技術只需要幾個小時，而且複雜性意味著只保留一個索引的『版本』。儘管可以進行一些修補以使新程式碼與舊索引對齊，但這仍然是一個有待解決的問題。

## Google」s Implementation

## 谷歌的實現

Google」s particular implementation of Code Search is tailored to the unique characteristics of its codebase, and the previous section outlined our design constraints for creating a robust and responsive index. The following section outlines how the Code Search team implemented and released its tool to Google developers.

Google 對程式碼搜尋的特殊實現是針對其程式碼庫的獨特特徵量身定製的，上一節概述了我們建立健壯且回應迅速的索引的設計約束。以下部分概述了程式碼搜尋團隊如何實施並向 Google 開發人員發布工具。

### Search Index

### 搜尋索引

Google」s codebase is a special challenge for Code Search due to its sheer size. In the early days, a trigram-based approach was taken. Russ Cox subsequently open sourced a simplified version. Currently, Code Search indexes about 1.5 TB of content and processes about 200 queries per second with a median server-side search latency of less than 50 ms and a median indexing latency (time between code commit and visibility in the index) of less than 10 seconds.

由於其龐大的規模，Google 的程式碼庫對程式碼搜尋來說是一個特殊的挑戰。在早期，採用了基於三元組的方法。 Russ Cox 隨後開源了一個簡化版本。目前，程式碼搜尋索引大約有1.5 TB 的內容，每秒處理大約 200 個查詢，伺服器端搜尋延遲的中位數小於 50 毫秒，索引延遲的中位數（程式碼送出和索引可見性之間的時間）小於 10秒。

Let」s roughly estimate the resource requirements to achieve this performance with a grep-based bruteforce solution. The RE2 library we use for regular expression matching processes about 100 MB/sec for data in RAM. Given a time window of 50 ms, 300,000 cores would be needed to crunch through the 1.5 TB of data. Because in most cases simple substring searches are sufficient, one could replace the regular expression matching with a special substring search that can process about 1 GB/sec[^11] under certain conditions, reducing the number of cores by 10 times. So far, we have looked at just the resource requirements for processing a single query within 50 ms. If we」re getting 200 requests per second, 10 of those will be simultaneously active in that 50 ms window, bringing us back to 300,000 cores just for substring search.

讓我們粗略估計一下使用基於 grep 的蠻力解決方案實現此效能所需的資源。我們用於正則表示式對應的 RE2 函式庫以大約 100 MB/秒的速度處理 RAM 中的資料。給定 50 毫秒的時間視窗，需要 300,000 個核心來處理 1.5 TB 的資料。因為在大多數情況下，簡單的子字串搜尋就足夠了，可以將正則表示式對應替換為特殊的子字串搜尋，在某些條件下可以處理大約 1 GB/秒，從而將核心數減少 10 倍。到目前為止，我們只研究了在 50 毫秒內處理單個查詢的資源需求。如果我們每秒收到 200 個請求，其中 10 個將在 50 毫秒的視窗中同時處於活動狀態，這使我們回到 300,000 個核心僅用於子字串搜尋。

Although this estimate ignores that the search can stop once a certain number of results are found or that file restrictions can be evaluated much more effectively than content searches, it doesn」t take communication overhead, ranking, or the fan out to tens of thousands of machines into account either. But it shows quite well the scale involved and why Google」s Code Search team continuously invests into improving indexing. Over the years, our index changed from the original trigram-based solution, through a custom suffix array–based solution, to the current sparse ngram solution. This latest solution is more than 500 times more efficient than the brute-force solution while being capable of also answering regular expression searches at blazing speed.

雖然這個估計忽略了一旦找到一定數量的結果，搜尋就會停止，或者檔案限制可以比內容搜尋更有效地評估，它不需要通訊開銷、排行或考慮數萬機器。它很好地展示了所涉及的巨大規模以及為什麼 Google 的程式碼搜尋團隊不斷投資於改進索引。多年來，我們的索引從最初的基於 trigram 的解決方案，透過基於自定義字尾陣列的解決方案，變為當前的稀疏 ngram 解決方案。這個最新的解決方案比蠻力解決方案的效率高出 500 多倍，同時還能夠以極快的速度回應正則表示式搜尋。

One reason we moved from a suffix array–based solution to a token-based n-gram solution was to take advantage of Google」s primary indexing and search stack. With a suffix array–based solution, building and distributing the custom indices becomes a challenge in and of itself. By utilizing 『standard』 technology, we benefit from all the advances in reverse index construction, encoding, and serving made by the core search team. Instant indexing is another feature that exists in standard search stacks, and by itself is a big challenge when solving it at scale.

我們從基於字尾陣列的解決方案轉向基於標記的 n-gram 解決方案的一個原因是利用 Google 的主要索引和搜尋堆疊。使用基於字尾陣列的解決方案，建立和分發自定義索引本身就是一項挑戰。透過利用『標準』技術，我們受益於核心搜尋團隊在反向索引建立、編碼和服務方面的進步。即時索引是標準搜尋堆疊中存在的另一個功能，在大規模解決它時，它本身就是一個巨大的挑戰。

Relying on standard technology is a trade-off between implementation simplicity and performance. Even though Google」s Code Search implementation is based on standard reverse indices, the actual retrieval, matching, and scoring are highly customized and optimized. Some of the more advanced Code Search features wouldn」t be possible otherwise. To index the history of file revisions, we came up with a custom compression scheme in which indexing the full history increased the resource consumption by a factor of just 2.5.

依賴標準技術是實現簡單性和效能之間的權衡。儘管 Google 的程式碼搜尋實現是基於標準的反向索引，但實際的檢索、對應和評分都是高度定製和最佳化的。否則，一些更高階的程式碼搜尋功能將無法實現。為了索引檔案修訂的歷史，我們提出了一個自定義壓縮方案，在該方案中，索引完整歷史將資源消耗增加了 2.5 倍。

In the early days, Code Search served all data from memory. With the growing index size, we moved the inverted index to flash. Although flash storage is at least an order of magnitude cheaper than memory, its access latency is at least two orders of magnitude higher. So, indices that work well in memory might not be suitable when served from flash. For instance, the original trigram index requires fetching not only a large number of reverse indices from flash, but also quite large ones. With n-gram schemes, both the number of inverse indices and their size can be reduced at the expense of a larger index.

在早期時候，程式碼搜尋從記憶體中提供所有資料。隨著索引大小的增加，我們將倒排索引移至快閃記憶體。儘管快閃記憶體儲存至少比記憶體便宜一個數量級，但它的訪問延遲至少要高兩個數量級。因此，在記憶體中執行良好的索引可能不適合從快閃記憶體提供服務。例如，原始的 trigram 索引不僅需要從快閃記憶體中獲取大量的反向索引，而且還需要相當大的索引。使用 n-gram 方案，可以以更大的索引為代價來減少逆索引的數量及其大小。

To support local workspaces (which have a small delta from the global repository), we have multiple machines doing simple brute-force searches. The workspace data is loaded on the first request and then kept in sync by listening for file changes. When we run out of memory, we remove the least recent workspace from the machines. The unchanged documents are searched with our history index. Therefore, the search is implicitly restricted to the repository state to which the workspace is synced.

為了支援本地工作空間（與全域儲存庫有一個小的增量），我們有多台機器進行簡單的暴力搜尋。工作區資料在第一次請求時載入，然後透過偵聽檔案更改來保持同步。當記憶體不足時，我們會從機器中刪除最近的工作區。使用我們的歷史索引搜尋未更改的文件。因此，搜尋被隱式限制為工作空間同步到的儲存庫狀態。

> [^11]: See https://blog.scalyr.com/2014/05/searching-20-gbsec-systems-engineering-before-algorithms and http://volnitsky.com/project/str_search./
> 11 查閱blog.scalyr.com/2014/05/searching-20-gbsec-systems-engineering-before-algorithms 和tp://volnitsky.com/project/str_search.

### Ranking

### 排行

For a very small codebase, ranking doesn」t provide much benefit, because there aren」t many results anyway. But the larger the codebase becomes, the more results will be found and the more important ranking becomes. In Google」s codebase, any short substring will occur thousands, if not millions, of times. Without ranking, the user either must check all of those results in order to find the correct one, or must refine the query[^12] er until the result set is reduced to just a handful of files. Both options waste the developer」s time.

對於非常小的程式碼庫，排行並沒有帶來太多好處，因為無論如何也沒有很多結果。但是程式碼庫越大，找到的結果就越多，排行也就越重要。在 Google 的程式碼庫中，任何短子字串都會出現數千次，甚至數百萬次。如果沒有排行，使用者要麼必須檢查所有這些結果才能找到正確的結果，要麼必須進一步細化查詢，直到結果集減少到幾個檔案。這兩種選擇都浪費了開發人員的時間。

Ranking typically starts with a scoring function, which maps a set of features of each file (『signals』) to some number: the higher the score, the better the result. The goal of the search is then to find the top N results as efficiently as possible. Typically, one distinguishes between two types of signals: those that depend only on the document (『query independent』) and those that depend on the search query and how it matches the document (『query dependent』). The filename length or the programming language of a file would be examples of query independent signals, whereas whether a match is a function definition or a string literal is a query dependent signal.

排行通常從評分函式開始，它將每個檔案的一組特徵（『訊號』）對映到某個數字：分數越高，結果越好。搜尋的目標是儘可能高效地找到前 N 個結果。通常，人們區分兩種型別的訊號：僅依賴於文件的訊號（『查詢無關』）和依賴於搜尋查詢以及它如何對應文件的訊號（『查詢依賴』）。檔案名長度或檔案的程式語言將是查詢獨立訊號的範例，而對應是函式定義還是字串文字是查詢相關訊號。

> [^12]: ontrast to web search, adding more characters to a Code Search query always reduces the result set (apart rom a few rare exceptions via regular expression terms).
>
> 12 與網路搜尋相比，在程式碼搜尋查詢中新增更多的字元總是會減少結果集（除了少數透過正則表示式術語的罕見例外）。

#### Query independent signals 查詢獨立訊號

Some of the most important query independent signals are the number of file views and the amount of references to a file. File views are important because they indicate which files developers consider important and are therefore more likely to want to find. For instance, utility functions in base libraries have a high view count. It doesn」t matter whether the library is already stable and isn」t changed anymore or whether the library is being actively developed. The biggest downside of this signal is the feedback loop it creates. By scoring frequently viewed documents higher, the chance increases that developers will look at them and decreases the chance of other documents to make it into the top N. This problem is known as exploitation versus exploration, for which various solutions exist (e.g., advanced A/B search experiments or curation of training data). In practice, it doesn」t seem harmful to somewhat over-show highscoring items: they are simply ignored when irrelevant and taken if a generic example is needed. However, it is a problem for new files, which don」t yet have enough information for a good signal.[^13]

一些最重要的獨立於查詢的訊號是檔案檢視的數量和對檔案的引用量。檔案檢視很重要，因為它們表明開發人員認為哪些檔案很重要，因此更有可能想要找到。例如，基礎庫中的實用程式函式具有很高的檢視次數。函式庫是否已經穩定並且不再更改或者函式庫是否正在積極開發都無關緊要。該訊號的最大缺點是它建立的反饋迴路。透過對經常檢視的文件進行更高的評分，開發人員檢視它們的機會增加，並降低了其他文件進入前 N 的機會。這個問題被稱為利用與探索，存在各種解決方案（例如，高階 A /B 搜尋實驗或訓練資料管理）。在實踐中，過度展示高分專案似乎並沒有什麼害處：它們在不相關時被忽略，如果需要通用範例則採用。但是，對於新檔案來說，這是一個問題，它們還沒有足夠的訊息來獲得良好的訊號。

We also use the number of references to a file, which parallels the original page rank algorithm, by replacing web links as references with the various kinds of 『include/import』 statements present in most languages. We can extend the concept up to build dependencies (library/module level references) and down to functions and classes. This global relevance is often referred to as the document」s 『priority.』

我們還使用檔案的引用數量，這與原始頁面排行演算法相似，透過將 Web 連結替換為大多數語言中存在的各種『包含/匯入』語句的引用。我們可以將概念向上擴充套件以建立依賴項（函式庫/模組級引用）並向下擴充套件至函式和類。這種全域相關性通常被稱為文件的『優先級』。

When using references for ranking, one must be aware of two challenges. First, you must be able to extract reference information reliably. In the early days, Google」s Code Search extracted include/import statements with simple regular expressions and then applied heuristics to convert them into full file paths. With the growing complexity of a codebase, such heuristics became error prone and challenging to maintain. Internally, we replaced this part with correct information from the Kythe graph.

在使用參考進行排行時，必須注意兩個挑戰。首先，你必須能夠可靠地提取參考訊息。早期，Google 的程式碼搜尋使用簡單的正則表示式提取包含/匯入語句，然後應用啟發式方法將它們轉換為完整的檔案路徑。隨著程式碼庫越來越複雜，這種啟發式方法變得容易出錯並且難以維護。在內部，我們用 Kythe 圖中的正確訊息替換了這部分。

Large-scale refactorings, such as open sourcing core libraries, present a second challenge. Such changes don」t happen atomically in a single code update; rather, they need to be rolled out in multiple stages. Typically, indirections are introduced, hiding, for example, the move of files from usages. These kinds of indirections reduce the page rank of moved files and make it more difficult for developers to discover the new location. Additionally, file views usually become lost when files are moved, making the situation even worse. Because such global restructurings of the codebase are comparatively rare (most interfaces move rarely), the simplest solution is to manually boost files during such transition periods. (Or wait until the migration completes and for the natural processes to up-rank the file in its new location.)

大規模重構，例如開源核心庫，是第二個挑戰。此類更改不會在單個程式碼更新中自動發生；相反，它們需要分多個階段推出。通常，引入間接方式，例如隱藏檔案的使用移動。這些型別間接降低了移動檔案的頁面排行，並使開發人員更難發現新位置。此外，移動檔案時文件檢視通常會丟失，從而使情況變得更糟。因為程式碼庫的這種全域重組比較少見（大多數介面很少移動），最簡單的解決方案是在這種轉場期間手動提升檔案。 （或者等到遷移完成並等待自然過程在其新位置對檔案進行升級。）

> [^13]: This could likely be somewhat corrected by using recency in some form as a signal, perhaps doing something imilar to web search dealing with new pages, but we don」t yet do so.
>
> 13 這很可能透過使用某種形式的事件作為訊號而得到一定程度的修正，也許可以做一些類似於網路搜尋處理新頁面的事情，但我們還沒有這樣做。

#### Query dependent signals 查詢相關訊號

Query independent signals can be computed offline, so computational cost isn」t a major concern, although it can be high. For example, for the 『page』 rank, the signal depends on the whole corpus and requires a MapReduce-like batch processing to calculate. Query dependent signals, which must be calculated for each query, should be cheap to compute. This means that they are restricted to the query and information quickly accessible from the index.

查詢獨立訊號可以離線計算，因此計算成本不是主要問題，儘管它可能很高。例如，對於『頁面』排行，訊號依賴於整個語料庫，需要類似 MapReduce 的批處理來計算。查詢相關訊號，即使必須為每個查詢進行計算，但是計算成本應該很低。這意味著它們僅限於從索引中快速訪問的查詢和訊息。

Unlike web search, we don」t just match on tokens. However, if there are clean token matches (that is, the search term matches with content with some form of breaks,such as whitespace, around it), a further boost is applied and case sensitivity is considered. This means, for example, a search for 『Point』 will score higher against "Point *p』 than against 『appointed to the council.』

與網路搜尋不同，我們不僅僅對應令牌。但是，如果存在乾淨的標記對應（即，搜尋詞與帶有某種形式的中斷（例如空格）的內容對應），則會應用進一步的提升並考慮區分大小寫。這意味著，例如，搜尋『Point』將針對『Point *p』的得分高於針對『被任命為理事會成員』的得分。

For convenience, a default search matches filename and qualified symbols[^14] ion to the actual file content. A user can specify the particular kind of match, but they don」t need to. The scoring boosts symbol and filename matches over normal content matches to reflect the inferred intent of the developer. Just as with web searches, developers can add more terms to the search to make queries more specific.It」s very common for a query to be 『qualified』 with hints about the filename (e.g.,『base』 or 『myproject』). Scoring leverages this by boosting results where much of the query occurs in the full path of the potential result, putting such results ahead of those that contain only the words in random places in their content.

為方便起見，除了實際檔案內容外，預設搜尋還對應檔案名和限定符號。使用者可以指定特定型別的對應，但他們不需要。與正常的內容對應相比，該評分提高了符號和檔案名對應，以反映開發人員的推斷意圖。與 Web 搜尋一樣，開發人員可以在搜尋中新增更多術語以使查詢更加具體。透過檔案名提示『限定』查詢是很常見的（例如，『基礎』或『我的專案』）。評分透過提升大部分查詢出現在潛在結果的完整路徑中的結果來利用這一點，將此類結果置於僅包含其內容中隨機位置的單詞的結果之前。

> [^14]: In programming languages, a symbol such as a function 『Alert』 often is defined in a particular scope, such as  class (『Monitor』) or namespace (『absl』). The qualified name might then be absl::Monitor::Alert, and this is indable, even if it doesn」t occur in the actual text.
>
> 14 在編程語言中，像函式 "Alert "這樣的符號經常被定義在一個特定的範圍內，例如類（"Monitor"）或命名空間（"absl"）。因此，限定的名稱可能是absl::Monitor::Alert，這是可以理解的，即使它沒有出現在實際文字中。

#### Retrieval 恢復

Before a document can be scored, candidates that are likely to match the search query are found. This phase is called retrieval. Because it is not practical to retrieve all documents, but only retrieved documents can be scored, retrieval and scoring must work well together to find the most relevant documents. A typical example is to search for a class name. Depending on the popularity of the class, it can have thousands of usages, but potentially only one definition. If the search was not explicitly restricted to class definitions, retrieval of a fixed number of results might stop before the file with the single definition was reached. Obviously, the problem becomes more challenging as the codebase grows.

在對文件進行評分之前，會找到可能與搜尋查詢對應的候選者。這個階段稱為檢索。因為檢索所有文件並不實用，只能對檢索到的文件進行評分，因此檢索和評分必須協同工作才能找到最相關的文件。一個典型的例子是搜尋類名。根據類別的受歡迎程度，它可以有數千種用法，但可能只有一種定義。如果搜尋沒有明確限制在類別定義中，則在到達具有單個定義的檔案之前，可能會停止檢索固定數量的結果。顯然，隨著程式碼庫的增長，問題變得更具挑戰性。

The main challenge for the retrieval phase is to find the few highly relevant files among the bulk of less interesting ones. One solution that works quite well is called supplemental retrieval. The idea is to rewrite the original query into more specialized ones. In our example, this would mean that a supplemental query would restrict the search to only definitions and filenames and add the newly retrieved documents to the output of the retrieval phase. In a naive implementation of supplemental retrieval, more documents need to be scored, but the additional partial scoring information gained can be used to fully evaluate only the most promising documents from the retrieval phase.

檢索階段的主要挑戰是在大量不那麼關聯的檔案中找到少數高度相關的檔案。一種效果很好的解決方案稱為補充檢索。這個方法是將原始查詢重寫為更專業的查詢。在我們的範例中，這意味著補充查詢會將搜尋限制為僅定義和檔案名，並將新檢索到的文件新增到檢索階段的輸出中。在補充檢索的簡單實現中，需要對更多文件進行評分，但獲得的額外部分評分訊息可用於全面評估檢索階段中最有希望的文件。

#### Result diversity 結果多樣性

Another aspect of search is diversity of results, meaning trying to give the best results in multiple categories. A simple example would be to provide both the Java and Python matches for a simple function name, rather than filling the first page of results with one or the other.

搜尋的另一個方面是結果的多樣性，這意味著試圖在多個類別中給出最好的結果。一個簡單的例子是為一個簡單的函式名提供 Java 和 Python 對應，而不是用一個或另一個填充結果的第一頁。

This is especially important when the intent of the user is not clear. One of the challenges with diversity is that there are many different categories—like functions,classes, filenames, local results, usages, tests, examples, and so on—into which results can be grouped, but that there isn」t a lot of space in the UI to show results for all of them or even all combinations, nor would it always be desirable. Google」s Code Search doesn」t do this as well as web search does, but the drop-down list of suggested results (like the autocompletions of web search) is tweaked to provide a diverse set of top filenames, definitions, and matches in the user」s current workspace.

當用戶的意圖不明確時，這一點尤其重要。多樣性的挑戰之一是有許多不同的類別—如函式、類、檔案名、本地結果、用法、測試、範例等—結果可以分組，但沒有很多UI 中的空間來顯示所有結果甚至所有組合的結果，這是不可取的。 Google 的程式碼搜尋在這方面的表現不如網路搜尋，但建議結果的下拉清單（如網路搜尋的自動完成）經過調整，可以對應使用者的當前工作區。

## Selected Trade-Offs 選擇的權衡

Implementing Code Search within a codebase the size of Google」s and keeping it responsive involved making a variety of trade-offs. These are noted in the following section.

在 Google 這麼大量級的程式碼庫中實現程式碼搜尋並保持其回應速度需要做出各種權衡。這些將在下一節中註明。

### Completeness: Repository at Head 完整性：倉庫頭部

We」ve seen that a larger codebase has negative consequences for search; for example, slower and more expensive indexing, slower queries, and noisier results. Can these costs be reduced by sacrificing completeness; in other words, leaving some content out of the index? The answer is yes, but with caution. Nontext files (binaries, images, videos, sound, etc.) are usually not meant to be read by humans and are dropped apart from their filename. Because they are huge, this saves a lot of resources. A more borderline case involves generated JavaScript files. Due to obfuscation and the loss of structure, they are pretty much unreadable for humans, so excluding them from the index is usually a good trade-off, reducing indexing resources and noise at the cost of completeness. Empirically, multimegabyte files rarely contain information relevant for developers, so excluding extreme cases is probably the correct choice.

我們已經看到，更大的程式碼庫會對搜尋產生負面影響；例如，更慢且更昂貴的索引、更慢的查詢和更嘈雜的結果。是否可以透過犧牲完整性來降低這些成本？換句話說，將一些內容排除在索引之外？答案是肯定的，但要謹慎。非文字檔案（二進位檔案、影象、影片、聲音等）通常不適合人類閱讀，而是從檔案名中刪除。因為它們很大，所以可以節省大量資源。更邊緣的情況涉及生成的 JavaScript 檔案。由於混淆和結構丟失，它們對人類來說幾乎是不可讀的，因此將它們從索引中排除通常是一個很好的權衡，以完整性為代價減少索引資源和噪音。根據經驗，數MB的檔案很少包含與開發人員相關的訊息，因此排除極端情況可能是正確的選擇。

However, dropping files from the index has one big drawback. For developers to rely on Code Search, they need to be able to trust it. Unfortunately, it is generally impossible to give feedback about incomplete search results for a specific search if the dropped files weren」t indexed in the first place. The resulting confusion and productivity loss for developers is a high price to pay for the saved resources. Even if developers are fully aware of the limitations, if they still need to perform their search, they will do so in an ad hoc and error-prone way. Given these rare but potentially high costs, we choose to err on the side of indexing too much, with quite high limits that are mostly picked to prevent abuse and guarantee system stability rather than to save resources.

但是，從索引中刪除檔案有一個很大的缺點。對於依賴程式碼搜尋的開發人員，他們需要能夠信任它。不幸的是，如果刪除的檔案一開始沒有被索引，通常不可能就特定搜尋的不完整搜尋結果提供反饋。給開發人員帶來的混亂和生產力損失是為節省的資源付出的高昂代價。即使開發人員完全意識到這些限制，如果他們仍然需要執行搜尋，他們也會以一種臨時且容易出錯的方式進行。鑑於這些罕見但潛在的高成本，我們選擇在索引過多方面犯錯，具有比較高的限制，是為了防止濫用和保證系統穩定性，而不是為了節省資源。

In the other direction, generated files aren」t in the codebase but would often be useful to index. Currently they are not, because indexing them would require integrating the tools and configuration to create them, which would be a massive source of complexity, confusion, and latency.

另一方面，生成的檔案不在程式碼庫中，但通常對索引很有用。雖然目前它們不是，是因為索引它們需要依賴整合工具和設定，這將是複雜性、混亂和延遲的巨大來源。

### Completeness: All Versus Most-Relevant Results 完整性：所有結果與最相關結果

Normal search sacrifices completeness for speed, essentially gambling that ranking will ensure that the top results will contain all of the desired results. And indeed, for Code Search, ranked search is the more common case in which the user is looking for one particular thing, such as a function definition, potentially among millions of matches. However, sometimes developers want all results; for example, finding all occurrences of a particular symbol for refactoring. Needing all results is common for analysis, tooling, or refactoring, such as a global search and replace. The need to deliver all results is a fundamental difference to web search in which many shortcuts can be taken, such as to only consider highly ranked items.

正常搜尋會犧牲完整性來換取速度，本質上是在賭排行會確保靠前的結果包含所有所需的結果。事實上，對於程式碼搜尋，排行搜尋是更常見的情況，例如使用者正在尋找一個特定的東西，函式定義，可能在數百萬個對應項中。但是，有時開發人員想要所有結果；例如，查詢特定符號的所有地方以進行重構。分析、工具或重構（例如全域搜尋和替換）通常需要所有結果。提供所有結果的需求是與 Web 搜尋之間的根本區別，其中可以採用許多捷徑，例如只考慮排行較高的專案。

Being able to deliver all results for very large result sets has high cost, but we felt it was required for tooling, and for developers to trust the results. However, because for most queries only a few results are relevant (either there are only a few matches[^15] or only a few are interesting), we didn」t want to sacrifice average speed for potential completeness.

能夠為非常大的結果集交付所有結果的成本很高，但我們認為這是工具所必需的，並且開發人員需要信任結果。然而，因為對於大多數查詢，只有少數結果是相關的（或者只有少數對應或只有少數是有用的），我們不想為了潛在的完整性而犧牲平均速度。

To achieve both goals with one architecture, we split the codebase into shards with files ordered by their priority. Then, we usually need to consider only the matches to high priority files from each chunk. This is similar to how web search works. However, if requested, Code Search can fetch all results from each chunk, to guarantee finding all results.[^16] This lets us address both use cases, without typical searches being slowed down by the less frequently used capability of returning large, complete results sets. Results can also then be delivered in alphabetical order, rather than ranked, which is useful for some tools.

為了透過一種架構實現這兩個目標，我們將程式碼庫拆分為分片，檔案按優先級排行。然後，我們通常只需要考慮每個塊中與高優先級檔案的對應。這類似於網路搜尋的工作方式。但是，如果需要，程式碼搜尋可以從每個塊中獲取所有結果，以保證找到所有結果。這讓我們能夠解決這兩個用例，而不會因為不常用的回傳大型完整結果集的功能而減慢典型搜尋速度。結果也可以按字母順序而不是排行，這對某些工具很有用。

So, here the trade-off was a more complex implementation and API versus greater capabilities, rather than the more obvious latency versus completeness.

因此，這裡權衡的是更復雜的實現和 API 與更強大的功能，而不是更明顯的延遲與完整性。

> [^15]: An analysis of queries showed that about one-third of user searches have fewer than 20 results.
>
> 15 對查詢的分析表明，大約三分之一的使用者搜尋結果少於20個。
>
> [^16]: In practice, even more happens behind the scenes so that responses don」t become painfully huge and developers don」t bring down the whole system by making searches that match nearly everything (imagine searching for the letter 『i』 or a single space).
>
> 16在實踐中，更多的事情發生在幕後，因此回應不會變得異常巨大，開發人員也不會透過搜尋幾乎所有內容來破壞整個系統（想象一下搜尋字母『i』或單個空格）

### Completeness: Head Versus Branches Versus All History Versus Workspaces 完整性：Head vs 分支 vs 所有歷史 vs 工作區

Related to the dimension of corpus size is the question of which code versions should be indexed: specifically, whether anything more than the current snapshot of code (『head』) should be indexed. System complexity, resource consumption, and overall cost increase drastically if more than a single file revision is indexed. To our knowledge, no IDE indexes anything but the current version of code. When looking at distributed version control systems like Git or Mercurial, a lot of their efficiency comes from the compression of their historical data. But the compactness of these representations becomes lost when constructing reverse indices. Another issue is that it is difficult to efficiently index graph structures, which are the basis for Distributed Version Control Systems.

與語料庫大小相關的是應該索引哪些程式碼版本的問題：具體來說，是否應該索引除當前程式碼快照（『head』）之外的任何內容。如果索引多個檔案修訂版，系統複雜性、資源消耗和總體成本會急劇增加。據我們所知，除了當前版本的程式碼之外，沒有任何 IDE 索引任何內容。在檢視像 Git 或 Mercurial 這樣的分散式版本控制系統時，它們的很多效率都來自對歷史資料的壓縮。但是在建立反向索引時，這些表示的緊湊性會丟失。另一個問題是很難有效地索引圖結構，這是分散式版本控制系統的基礎。

Although it is difficult to index multiple versions of a repository, doing so allows the exploration of how code has changed and finding deleted code. Within Google, Code Search indexes the (linear) Piper history. This means that the codebase can be searched at an arbitrary snapshot of the code, for deleted code, or even for code authored by certain people.

儘管索引儲存庫的多個版本很困難，但這樣做可以探索程式碼如何更改並找到已刪除的程式碼。在 Google 中，程式碼搜尋索引（線性）Piper 歷史。這意味著可以在程式碼的任意快照中搜索程式碼庫，查詢已刪除的程式碼，甚至是某些人創作的程式碼。

One big benefit is that obsolete code can now simply be deleted from the codebase. Before, code was often moved into directories marked as obsolete so that it could still be found later. The full history index also laid the foundation for searching effectively in people」s workspaces (unsubmitted changes), which are synced to a specific snapshot of the codebase. For the future, a historical index opens up the possibility of interesting signals to use when ranking, such as authorship, code activity, and so on. Workspaces are very different from the global repository:

- Each developer can have their own workspaces.
- There are usually a small number of changed files within a workspace.
- The files being worked on are changing frequently.
- A workspace exists only for a relatively short time period.

一個大的優點是現在可以簡單地從程式碼庫中刪除過時的程式碼。以前，程式碼經常被移動到標記為過時的目錄中，以便以後仍然可以找到它。完整的歷史索引還為在人們的工作空間（未送出的更改）中進行有效搜尋奠定了基礎，這些工作空間與程式碼庫的特定快照同步。對於未來，歷史索引開闢了在排行時使用有效訊號的可能性，例如作者身份、程式碼活動等。工作區與全域儲存庫有很大不同：

- 每個開發人員都可以擁有自己的工作區。
- 工作空間中通常有少量更改的檔案。
- 正在處理的檔案經常更改。
- 工作空間僅存在相對較短的時間段。

To provide value, a workspace index must reflect exactly the current state of the workspace.

為了提供價值，工作區索引必須準確反映工作區的當前狀態。

### Expressiveness: Token Versus Substring Versus Regex 表現力：令牌與子字串與正則表示式

The effect of scale is greatly influenced by the supported search feature set. Code Search supports regular expression (regex) search, which adds power to the query language, allowing whole groups of terms to be specified or excluded, and they can be used on any text, which is especially helpful for documents and languages for which deeper semantic tools don」t exist.

規模的效果受到支援的搜尋特徵集的很大影響。程式碼搜尋支援正則表示式 (regex) 搜尋，這增加了查詢語言的功能，允許指定或排除整組術語，並且它們可以用於任何文字，在不存在更深層次的語義工具的情況下，對於文件和語言特別有用。

Developers are also used to using regular expressions in other tools (e.g., grep) and contexts, so they provide powerful search without adding to a developer」s cognitive load. This power comes at a cost given that creating an index to query them efficiently is challenging. What simpler options exist?

開發人員還習慣於在其他工具（例如 grep）和上下文中使用正則表示式，因此它們提供了強大的搜尋功能，而不會增加開發人員的認知負擔。鑑於建立索引以有效地查詢它們具有挑戰性，因此這種能力是有代價的。有哪些更簡單的選擇？

A token-based index (i.e., words) scales well because it stores only a fraction of the actual source code and is well supported by standard search engines. The downside is that many use cases are tricky or even impossible to realize efficiently with a tokenbased index when dealing with source code, which attaches meaning to many characters typically ignored when tokenizing. For example, searching for 『function()』 versus 『function(x)』, 『(x ^ y)』, or 『=== myClass』 is difficult or impossible in most token-based searches.

基於標記的索引（例如：單詞）可以很好地擴充套件，因為它只儲存實際原始碼的一小部分，並且得到標準搜尋引擎的良好支援。不利的一面是，在處理原始碼時，使用基於標記的索引來有效地實現許多用例是棘手的，甚至不可能有效地實現，這為標記化時通常被忽略的許多字元附加了意義。例如，在大多數基於標記的搜尋中，搜尋『function()』與『function(x)』、『(x ^ y)』或『=== myClass』是困難的或不可能的。

Another problem of tokenization is that tokenization of code identifiers is ill defined. Identifiers can be written in many ways, such as CamelCase, snake_case, or even justmashedtogether without any word separator. Finding an identifier when remembering only some of the words is a challenge for a token-based index.

標記化的另一個問題是程式碼識別符號的標記化定義不明確。識別符號可以用多種方式編寫，例如 CamelCase、snake_case，甚至只是混合在一起而無需任何單詞分隔符。在只記住一些單詞時找到一個識別符號對於基於標記的索引來說是一個挑戰。

Tokenization also typically doesn」t care about the case of letters (『r』 versus 『R』), and will often blur words; for example, reducing 『searching』 and 『searched』 to the same stem token search. This lack of precision is a significant problem when searching code. Finally, tokenization makes it impossible to search on whitespace or other word delimiters (commas, parentheses), which can be very important in code.

標記化通常也不關心字母的大小寫（『r』與『R』），並且經常會模糊單詞；例如，將『searching』和『searched』簡化為相同的詞幹標記搜尋。在搜尋程式碼時，缺乏精確性是一個嚴重的問題。最後，標記化使搜尋空格或其他單詞分隔符（逗號、括號）成為不可能，即使這在程式碼中可能非常重要。

A next step up[^17] in searching power is full substring search in which any sequence of characters can be searched for. One fairly efficient way to provide this is via a trigram-based index. [^18] In its simplest form, the resulting index size is still much smaller than the original source code size. However, the small size comes at the cost of relatively low recall accuracy compared to other substring indices. This means slower queries because the nonmatches need to be filtered out of the result set. This is where a good compromise between index size, search latency, and resource consumption must be found that depends heavily on codebase size, resource availability, and searches per second.

搜尋能力的下一步是完整的子字串搜尋，其中可以搜尋任何字元序列。提供此功能的一種相當有效的方法是透過基於三元組的索引。在最簡單的形式中，生成的索引大小仍然比原始碼大小小得多。然而，與其他子字串索引相比，小尺寸的代價是召回準確率相對較低。這意味著查詢速度較慢，因為不對應項需要從結果集中過濾掉。這是必須在索引大小、搜尋延遲和資源消耗之間找到良好折衷的地方，這在很大程度上取決於程式碼庫大小、資源可用性和每秒搜尋量。

If a substring index is available, it」s easy to extend it to allow regular expression searches. The basic idea is to convert the regular expression automaton into a set of substring searches. This conversion is straightforward for a trigram index and can be generalized to other substring indices. Because there is no perfect regular expression index, it will always be possible to construct queries that result in a brute-force search. However, given that only a small fraction of user queries are complex regular expressions, in practice, the approximation via substring indices works very well.

如果子字串索引可用，很容易擴充套件它以允許正則表示式搜尋。基本思想是將正則表示式自動機轉換為一組子字串搜尋。這種轉換對於三元索引很簡單，並且可以推廣到其他子字串索引。因為沒有完美的正則表示式索引，所以總是可以建立導致暴力搜尋的查詢。然而，鑑於只有一小部分使用者查詢是複雜的正則表示式，在實踐中，透過子字串索引的近似效果非常好。

> [^17]: There are other intermediate varieties, such as building a prefix/suffix index, but generally they provide less expressiveness in search queries while still having high complexity and indexing costs.
>
> 17 還有其他的類似方式，如建立字首/字尾索引，但一般來說，它們在搜尋查詢中提供的表達能力較低，同時仍有較高的複雜性和索引成本。
> 
[^18]: Russ Cox, 『Regular Expression Matching with a Trigram Index or How Google Code Search Worked.』
>
> 18 Russ Cox，"用三元索引進行正則表示式對應或谷歌程式碼搜尋的工作原理"。

## Conclusion 結論

Code Search grew from an organic replacement for grep into a central tool boosting developer productivity, leveraging Google」s web search technology along the way. What does this mean for you, though? If you are on a small project that easily fits in your IDE, probably not much. If you are responsible for the productivity of engineers on a larger codebase, there are probably some insights to be gained.

程式碼搜尋從 grep 的有機替代品發展成為提高開發人員生產力的核心工具，並在此過程中利用了 Google 的網路搜尋技術。不過，這對你意味著什麼？如果你在一個很容易融入你的 IDE 的小專案上，可能不多。如果你負責在更大的程式碼庫上提高工程師的生產力，那麼你可能會獲得一些見解。

The most important one is perhaps obvious: understanding code is key to developing and maintaining it, and this means that investing in understanding code will yield dividends that might be difficult to measure, but are real. Every feature we added to Code Search was and is used by developers to help them in their daily work (admittedly some more than others). Two of the most important features, Kythe integration (i.e., adding semantic code understanding) and finding working examples, are also the most clearly tied to understanding code (versus, for example, finding it, or seeing how it」s changed). In terms of tool impact, no one uses a tool that they don」t know exists, so it is also important to make developers aware of the available tooling—at Google, it is part of 『Noogler』 training, the onboarding training for newly hired software engineers.

最重要的一點可能是顯而易見的：理解程式碼是開發和維護程式碼的關鍵，這意味著投資在理解程式碼上將產生可能難以衡量但實實在在的紅利。我們新增到程式碼搜尋中的每個功能都被開發人員用來幫助他們完成日常工作（誠然，其中一些功能比其他功能更多）。兩個最重要的功能，Kythe 整合（即新增語義程式碼理解）和查詢工作範例，也與理解程式碼最明顯相關（例如，查詢程式碼或檢視程式碼如何更改）。就工具影響而言，沒有人使用他們不知道存在的工具，因此讓開發人員瞭解可用工具也很重要——在谷歌，它是『Noogler』培訓的一部分，即新人的入職培訓和聘請的軟體工程師培訓。

For you, this might mean setting up a standard indexing profile for IDEs, sharing knowledge about egrep, running ctags, or setting up some custom indexing tooling, like Code Search. Whatever you do, it will almost certainly be used, and used more, and in different ways than you expected—and your developers will benefit.

對你而言，這可能意味著為 IDE 設定標準索引設定檔案、分享有關 egrep 的知識、執行 ctags 或設定一些自定義索引工具，例如程式碼搜尋。無論你做什麼，它幾乎肯定會被使用，而且使用得更多，而且使用的方式與你預期的不同—你的開發人員將從中受益。

## TL;DRs  內容提要

- Helping your developers understand code can be a big boost to engineering productivity. At Google, the key tool for this is Code Search.
- Code Search has additional value as a basis for other tools and as a central, standard place that all documentation and developer tools link to.
- The huge size of the Google codebase made a custom tool—as opposed to, for example, grep or an IDE」s indexing—necessary.
- As an interactive tool, Code Search must be fast, allowing a 『question and answer』 workflow. It is expected to have low latency in every respect: search, browsing, and indexing.
- t will be widely used only if it is trusted, and will be trusted only if it indexes all code, gives all results, and gives the desired results first. However, earlier, less powerful, versions were both useful and used, as long as their limits were understood.

- 幫助你的開發人員理解程式碼可以大大提高工程生產力。在 Google，這方面的關鍵工具是程式碼搜尋。
- 作為其他工具的基礎以及作為所有文件和開發工具連線到的中心標準位置，程式碼搜尋具有附加價值
- Google 程式碼庫的龐大規模使得定製工具（例如，與 grep 或 IDE 的索引不同）成為必要。
- 作為一種互動式工具，程式碼搜尋必須快速，允許『問題和回答』的工作流程。預計在各個方面都有低延遲：搜尋，瀏覽和索引。
- 只有當它被信任時才會被廣泛使用，並且只有當它索引所有程式碼、給出所有結果並首先給出期望的結果時才會被信任。但是，只要瞭解其侷限性，較早的、功能較弱的版本既有用又可以使用。
