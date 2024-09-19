
**CHAPTER 20**

# Static Analysis
# 第二十章 靜態分析
        
**Written by Caitlin Sadowski**

**Edited by Lisa Carey**

Static analysis refers to programs analyzing source code to find potential issues such as bugs, antipatterns, and other issues that can be diagnosed *without executing the* *program*. The 『static』 part specifically refers to analyzing the source code instead of a running program (referred to as 『dynamic』 analysis). Static analysis can find bugs in programs early, before they are checked in as production code. For example, static analysis can identify constant expressions that overflow, tests that are never run, or invalid format strings in logging statements that would crash when executed.[^1] However, static analysis is useful for more than just finding bugs. Through static analysis at Google, we codify best practices, help keep code current to modern API versions, and prevent or reduce technical debt. Examples of these analyses include verifying that naming conventions are upheld, flagging the use of deprecated APIs, or pointing out simpler but equivalent expressions that make code easier to read. Static analysis is also an integral tool in the API deprecation process, where it can prevent backsliding during migration of the codebase to a new API (see Chapter 22). We have also found evidence that static analysis checks can educate developers and actually prevent antipatterns from entering the codebase.[^2]

靜態分析是指透過程式分析原始碼來發現潛在的問題，例如bug、反模式和其他無需執行程式就能發現的問題。『靜態』具體是指分析原始碼，而不是執行中的程式（即『動態』分析）。它可以在程式碼被合入生產環境前發現bug，例如，可以識別溢位的常數表示式、永遠不會執行的測試用例或日誌字串的無效格式化導致執行崩潰的問題。但靜態分析的作用不只是查詢bug。透過對Google程式碼的靜態分析，我們編寫了最佳實踐，幫助推進程式碼使用最新介面和減少技術債，這些分析的例子包括：校驗是否遵循命名規範；標記已棄用但仍然使用的介面；簡化表示式以提高程式碼可讀性。靜態分析也是棄用某個介面時不可或缺的工具，它可以防止將程式碼庫遷移到新介面時出現『倒退』現象（參見第22章，指被呼叫系統不斷遷移舊介面到新介面，而其他系統不斷的呼叫棄用介面而不呼叫新介面）。我們還發現靜態分析檢查可以對開發人員起到啟發和約束作用，可以防止開發人員寫出反模式的程式碼。

In this chapter, we」ll look at what makes effective static analysis, some of the lessons we at Google have learned about making static analysis work, and how we implemented these best practices in our static analysis tooling and processes.[^3]

本章我們將介紹如何進行有效的靜態分析，包含我們在 Google 瞭解到的一些關於靜態分析工作的經驗和我們在靜態分析工具和流程中的最佳實踐。

> [^1]: See `http://errorprone.info/bugpatterns`.
>
> 1 查閱 `http://errorprone.info/bugpatterns`。
>
> [^2]: Caitlin Sadowski et al. Tricorder: Building a Program Analysis Ecosystem, International Conference on Software Engineering (ICSE), May 2015.
>
> 2 Caitlin Sadowski等人，Tricorder。建立一個程式分析生態系統，國際軟體工程會議（ICSE），2015年5月。
>
> [^3]: A good academic reference for static analysis theory is: Flemming Nielson et al. Principles of Program Analysis (Gernamy: Springer, 2004)
>
> 3 關於靜態分析理論，一個很好的學術參考資料是。Flemming Nielson等人，《程式分析原理》(Gernamy: Springer, 2004)

## 有效靜態分析的特點

Although there have been decades of static analysis research focused on developing new analysis techniques and specific analyses, a focus on approaches for improving *scalability* and *usability* of static analysis tools has been a relatively recent development.

儘管幾十年來，靜態分析研究一直專注於開發新的分析技術和具體分析，但提高靜態分析工具的可擴充性和可用性的方法最近才開始發展。

### Scalability  可擴充性

Because modern software has become larger, analysis tools must explicitly address scaling in order to produce results in a timely manner, without slowing down the software development process. Static analysis tools at Google must scale to the size of Google」s multibillion-line codebase. To do this, analysis tools are shardable and incremental. Instead of analyzing entire large projects, we focus analyses on files affected by a pending code change, and typically show analysis results only for edited files or lines. Scaling also has benefits: because our codebase is so large, there is a lot of low- hanging fruit in terms of bugs to find. In addition to making sure analysis tools can run on a large codebase, we also must scale up the number and variety of analyses available. Analysis contributions are solicited from throughout the company. Another component to static analysis scalability is ensuring the *process* is scalable. To do this, Google static analysis infrastructure avoids bottlenecking analysis results by showing them directly to relevant engineers.

現代軟體變得越來越大，為了使分析工具在不減慢軟體開發過程的情況下及時生效，必須有效地解決擴充性問題。對 Google 來說，分析工具需要滿足 Google 數十億行程式碼函式庫的規模。為此，分析工具是分片和增量分析的，即不是分析整個大型專案，而是將分析重點放在受待處理程式碼變更影響的檔案上，並且通常僅顯示已編輯檔案或行的分析結果。擴充套件也有好處：因為我們的程式碼庫非常大，這樣做在尋找bug時容易的多。除了確保分析工具可以在大型程式碼庫上執行之外，還需要必須擴大可分析的數量和種類。分析器是在整個公司範圍內徵求的。靜態分析可擴充性的另一個組成部分是確保過程是可擴充套件的，為此，Google靜態分析基礎架構透過直接向相關工程師展示分析結果來避免造成分析瓶頸。

### Usability  可用性

When thinking about analysis usability, it is important to consider the cost-benefit trade-off for static analysis tool users. This 『cost』 could either be in terms of developer time or code quality. Fixing a static analysis warning could introduce a bug. For code that is not being frequently modified, why 『fix』 code that is running fine in production? For example, fixing a dead code warning by adding a call to the previously dead code could result in untested (possibly buggy) code suddenly running. There is unclear benefit and potentially high cost. For this reason, we generally focus on newly introduced warnings; existing issues in otherwise working code are typically only worth highlighting (and fixing) if they are particularly important (security issues, significant bug fixes, etc.). Focusing on newly introduced warnings (or warnings on modified lines) also means that the developers viewing the warnings have the most relevant context on them.

考慮可用性時，重要的是要考慮靜態分析工具使用者的成本效益權衡。這種』成本』可能是開發時間或程式碼品質。修復靜態分析警告可能會引入錯誤的。對於不經常被修改的程式碼，為什麼要 "修復 "在生產中執行良好的程式碼？例如，透過新增對死程式碼(從未被執行過的程式碼)的呼叫來修復硬編碼警告，可能會導致未經測試（可能有錯誤）的程式碼突然執行。這種做法收益不明確，但是成本可能很高。出於這個原因，我們通常只關注新引入的警告，程式碼中的現有問題通常只在特別重要（安全問題、重大錯誤修復等）時才值得修復。關注新引入的警告（或修改行上的警告）也意味著檢視警告的開發人員具有最相關的上下文和背景。

Also, developer time is valuable! Time spent triaging analysis reports or fixing highlighted issues is weighed against the benefit provided by a particular analysis. If the analysis author can save time (e.g., by providing a fix that can be automatically applied to the code in question), the cost in the trade-off goes down. Anything that can be fixed automatically should be fixed automatically. We also try to show developers reports about issues that actually have a negative impact on code quality so that they do not waste time slogging through irrelevant results.

此外，開發人員的時間很寶貴，要對分析報告進行分類或修復突出問題所花費的時間與特定分析提供的收益進行權衡。如果分析可以節省時間（例如，透過提供可以自動應用於相關程式碼的修復），則成本就會下降。任何可以自動修復的東西都應該自動修復。我們還嘗試向開發人員展示實際上對程式碼品質有負面影響的問題的報告，這樣他們就不會浪費時間費力地處理不相關的分析結果。

To further reduce the cost of reviewing static analysis results, we focus on smooth developer workflow integration. A further strength of homogenizing everything in one workflow is that a dedicated tools team can update tools along with workflow and code, allowing analysis tools to evolve with the source code in tandem.

為了進一步降低檢視靜態分析結果的成本，我們將重點放在平滑的開發人員工作流程整合上。在一個工作流中同質化所有內容的另一個優勢是，一個專門的工具團隊可以隨著工作流和程式碼一起更新工具，從而允許分析工具與原始碼同步發展。

We believe these choices and trade-offs that we have made in making static analyses scalable and usable arise organically from our focus on three core principles, which we formulate as lessons in the next section.

我們在使靜態分析具有可擴充性和可用性方面所做的這些選擇和權衡，是從我們對三個核心原則的關注中產生的，我們將在下一節中闡述這三個原則作為經驗教訓。

## Key Lessons in Making Static Analysis Work 靜態分析工作中的關鍵工作

There are three key lessons that we have learned at Google about what makes static analysis tools work well. Let」s take a look at them in the following subsections.

我們在 Google 瞭解到瞭如何用好靜態分析工具的三個關鍵點。讓我們在下面的小節中看看它們。

### Focus on Developer Happiness  關注開發者的幸福感

We mentioned some of the ways in which we try to save developer time and reduce the cost of interacting with the aforementioned static analysis tools; we also keep track of how well analysis tools are performing. If you don」t measure this, you can」t fix problems. We only deploy analysis tools with low false-positive rates (more on that in a minute). We also *actively solicit and act on feedback* from developers consuming static analysis results, in real time. Nurturing this feedback loop between static analysis tool users and tool developers creates a virtuous cycle that has built up user trust and improved our tools. User trust is extremely important for the success of static analysis tools.

我們提到了一些試圖節省開發人員時間並降低與靜態分析工具互動成本的方法，我們還跟蹤分析工具的效能。如果你不衡量這點，你就無法解決問題。我們只部署誤報率較低的分析工具（稍後將詳細介紹）。我們還*積極徵求開發人員對靜態分析結果的實時反饋並採取行動*，在靜態分析工具使用者和開發人員之間形成反饋閉環，創造一個良性迴圈，建立了使用者信任，藉此改進我們的工具。使用者信任對於靜態分析工具的成功至關重要。

For static analysis, a 『false negative』 is when a piece of code contains an issue that the analysis tool was designed to find, but the tool misses it. A 『false positive』 occurs when a tool incorrectly flags code as having the issue. Research about static analysis tools traditionally focused on reducing false negatives; in practice, low false-positive rates are often critical for developers to actually want to use a tool—who wants to wade through hundreds of false reports in search of a few true ones?[^4]

對於靜態分析，『漏報（false negative）』是指一段程式碼包含分析工具找到的問題，但該工具忽略了該問題，『誤報（false positive）』是指工具錯誤地將程式碼標記為存在問題。一般來說，靜態分析工具的研究側重於減少誤判；實踐中，開發者是否真正想要使用工具取決於誤報率是否很低——誰願意在數百個虛假報告中費力尋找一些真實的報告？

Furthermore, perception is a key aspect of the false-positive rate. If a static analysis tool is producing warnings that are technically correct but misinterpreted by users as false positives (e.g., due to confusing messages), users will react the same as if those warnings were in fact false positives. Similarly, warnings that are technically correct but unimportant in the grand scheme of things provoke the same reaction. We call the user-perceived false-positive rate the 『effective false positive』 rate. An issue is an 『effective false positive』 if developers did not take some positive action after seeing the issue. This means that if an analysis incorrectly reports an issue, yet the developer happily makes the fix anyway to improve code readability or maintainability, that is not an effective false positive. For example, we have a Java analysis that flags cases in which a developer calls the contains method on a hash table (which is equivalent to containsValue) when they actually meant to call containsKey—even if the developer correctly meant to check for the value, calling containsValue instead is clearer. Similarly, if an analysis reports an actual fault, yet the developer did not understand the fault and therefore took no action, that is an effective false positive.

此外，使用者感知是誤報率的一個關鍵方面。如果靜態分析工具產生的警告在技術上是正確的，但被使用者誤解為誤報（例如，由於告警訊息混亂），使用者的反應將與這些警告實際上是誤報一樣。類似地，技術上正確但在大局中不重要的警告也會引發同樣的反應。我們將使用者感知的誤報率稱為『有效誤報率』。如果開發者在看到問題後沒有採取積極的行動，那麼問題就是『有效的誤報（effective false positive）』，這意味著，如果一個分析錯誤地報告了一個問題，但開發人員仍然樂於進行修復，以提高程式碼的可讀性或可維護性，那麼這就不是一個有效的誤報。例如，我們有一個Java分析，它標記了這樣一種情況：當開發人員實際上打算呼叫containsKey時，開發人員在雜湊表（相當於containsValue）上呼叫contains方法，即使開發人員正確地打算檢查值，呼叫containsValue反而更清晰。同樣，如果分析報告了一個實際的故障，但開發人員不瞭解故障，因此沒有採取任何行動，這就是一個有效的誤報。

> [^4]: Note that there are some specific analyses for which reviewers might be willing to tolerate a much higher false-positive rate: one example is security analyses that identify critical problems.
>
> 4 請注意，有一些特定的分析，審查員可能願意容忍更高的誤報率：一個例子是識別關鍵問題的安全分析。

### Make Static Analysis a Part of the Core Developer Workflow  使靜態分析成為核心開發人員工作流程的一部分

At Google, we integrate static analysis into the core workflow via integration with code review tooling. Essentially all code committed at Google is reviewed before being committed; because developers are already in a change mindset when they send code for review, improvements suggested by static analysis tools can be made without too much disruption. There are other benefits to code review integration. Developers typically context switch after sending code for review, and are blocked on reviewers— there is time for analyses to run, even if they take several minutes to do so. There is also peer pressure from reviewers to address static analysis warnings. Furthermore, static analysis can save reviewer time by highlighting common issues automatically; static analysis tools help the code review process (and the reviewers) scale. Code review is a sweet spot for analysis results.[^5]

在 Google ，我們透過與程式碼審查工具整合，將靜態分析整合到核心工作流中。基本上 Google 送出的所有程式碼在送出之前都會經過審查，因為開發人員在傳送程式碼供審查時已經改變了心態，所以靜態分析工具建議的改進可以在沒有太多干擾的情況下進行。程式碼審查整合還有其他好處，開發人員通常在傳送程式碼進行審查後切換上下文，並且在審查員面前被阻止——即使需要幾分鐘的時間來執行分析。來自審查員的同行壓力也要求解決靜態分析警告問題，此外，靜態分析可以自動突出常見問題，從而節省審閱者的時間，這有助於程式碼評審過程（以及審查員）的規模化。程式碼評審是分析結果的最佳選擇。

> [^5]: See later in this chapter for more information on additional integration points when editing and browsing code.
>
> 5 關於編輯和瀏覽程式碼時的額外整合點的更多訊息，請參見本章後面的內容。

### Empower Users to Contribute  允許使用者做出貢獻

There are many domain experts at Google whose knowledge could improve code produced. Static analysis is an opportunity to leverage expertise and apply it at scale by having domain experts write new analysis tools or individual checks within a tool.

Google有許多領域專家，他們的知識可以改進生成的程式碼。靜態分析創造了一個利用他們的專業知識並大規模應用的機會，即利用領域專家編寫新的分析工具或在工具中進行單獨檢查。

For example, experts who know the context for a particular kind of configuration file can write an analyzer that checks properties of those files. In addition to domain experts, analyses are contributed by developers who discover a bug and would like to prevent the same kind of bug from reappearing anywhere else in the codebase. We focus on building a static analysis ecosystem that is easy to plug into instead of integrating a small set of existing tools. We have focused on developing simple APIs that can be used by engineers throughout Google—not just analysis or language experts— to create analyses; for example, Refaster[^6] enables writing an analyzer by specifying pre- and post-code snippets demonstrating what transformations are expected by that analyzer.

例如，瞭解特定型別設定檔案上下文的專家可以編寫一個分析器來檢查這些檔案的屬性。除了領域專家之外，發現bug並希望防止同類bug在程式碼庫中的任何其他地方再次出現的開發人員也可以提供貢獻。我們專注於建立一個易於插入的靜態分析生態系統，而不是整合一小部分現有工具。我們專注於開發簡單的API，可供整個 Google 的工程師（不僅僅是分析或語言專家）用來建立分析；例如，重構可以透過指定前後程式碼片段來編寫分析器，來達到該分析器期望的效果。

> [^6]: Louis Wasserman, 『Scalable, Example-Based Refactorings with Refaster.』 Workshop on Refactoring Tools, 2013.
>
> 6 Louis Wasserman，"用Refaster進行可擴充套件的、基於實例的重構"。重構工具研討會，2013年。

## Tricorder: Google」s Static Analysis Platform  Tricorder： Google 的靜態分析平台

Tricorder, our static analysis platform, is a core part of static analysis at Google.[^7] Tricorder came out of several failed attempts to integrate static analysis with the developer workflow at Google;[^8] the key difference between Tricorder and previous attempts was our relentless focus on having Tricorder deliver only valuable results to its users. Tricorder is integrated with the main code review tool at Google, Critique. Tricorder warnings show up on Critique」s diff viewer as gray comment boxes, as demonstrated in Figure 20-1.

我們的靜態分析平台 Tricorder是Google靜態分析的核心部分。Tricorder是在Google多次嘗試將靜態分析與開發人員工作流整合的失敗嘗試中誕生的，與之前嘗試的主要區別在於我們堅持不懈地致力於讓Tricorder只為使用者提供有價值的結果。Tricorder與 Google 的主要程式碼審查工具Critique整合在一起。 Tricorder警告在Critique的差異檢視器上顯示為灰色的註解框，如圖 20-1 所示。

![Figure 20-1](./images/Figure%2020-1.png)

*Figure 20-1. Critique」s diff viewing, showing a static analysis warning from Tricorder in* *gray*  圖20-1. Critique的diff檢視，灰色顯示了Tricorder的靜態分析警告

To scale, Tricorder uses a microservices architecture. The Tricorder system sends analyze requests to analysis servers along with metadata about a code change. These servers can use that metadata to read the versions of the source code files in the change via a FUSE-based filesystem and can access cached build inputs and outputs. The analysis server then starts running each individual analyzer and writes the output to a storage layer; the most recent results for each category are then displayed in Critique. Because analyses sometimes take a few minutes to run, analysis servers also post status updates to let change authors and reviewers know that analyzers are running and post a completed status when they have finished. Tricorder analyzes more than 50,000 code review changes per day and is often running several analyses per second.

為了方便擴充套件，Tricorder使用微服務架構。 Tricorder系統將分析請求連同有關程式碼更改的元資料傳送到分析伺服器。這些伺服器可以使用該元資料透過基於FUSE的檔案系統讀取更改中原始碼檔案的版本，並且可以訪問快取的建立輸入和輸出。然後分析伺服器開始執行每個單獨的分析器並將輸出寫入儲存層。每個類別的最新結果隨後會顯示在Critique中。因為分析有時需要等幾分鐘，分析伺服器也會發布狀態更新，讓程式碼作者和審查員知道分析器正在執行，並在完成後發布完成狀態。Tricorder每天分析超過50,000次程式碼審查更改，並且通常每秒執行多次分析。整個Google的開發人員編寫Tricorder分析（稱為『分析器』）或為現有分析貢獻單獨的『檢查』。

Developers throughout Google write Tricorder analyses (called 『analyzers』) or contribute individual 『checks』 to existing analyses. There are four criteria for new Tricorder checks:

- *Be understandable*  
    Be easy for any engineer to understand the output.
- *Be* *actionable* *and* *easy* *to* *fix*  
    The fix might require more time, thought, or effort than a compiler check, and the result should include guidance as to how the issue might indeed be fixed.
- *Produce less than 10% effective false positives*  
    Developers should feel the check is pointing out an actual issue [at least 90% of](https://oreil.ly/ARSzt) [the time](https://oreil.ly/ARSzt).
- *Have* *the potential for significant impact on code quality*  
    The issues might not affect correctness, but developers should take them seriously and deliberately choose to fix them.

Tricorder 檢查有四個標準：

- *易於理解*  
​    任何工程師都可以輕鬆理解輸出結果。
- *可操作且易於修復*  
​    與編譯器檢查相比，修復可能需要更多的時間、思考或嘗試，結果應包括有關如何真正修復問題的指導。
- *少於10%的有效誤報*  
​    開發人員應該覺得檢查至少在90%的時間裡指出了實際問題。
- *有可能對程式碼品質產生重大影響*  
​    這些問題可能不會影響正確性，但開發人員應該認真對待它們並有意識地選擇修復它們。

Tricorder analyzers report results for more than 30 languages and support a variety of analysis types. Tricorder includes more than 100 analyzers, with most being contributed from outside the Tricorder team. Seven of these analyzers are themselves plug-in systems that have hundreds of additional checks, again contributed from developers across Google. The overall effective false-positive rate is just below 5%.

Tricorder分析儀報告支援30種語言，並支援多種分析型別。Tricorder包括100多個分析器，其中大部分來自Tricorder團隊外部。 其中七個分析器本身就是外掛系統，具有數百項額外檢查，由 Google 的開發人員提供，總體有效的誤報率略低於 5%。

> [^7]: Caitlin Sadowski, Jeffrey van Gogh, Ciera Jaspan, Emma Söderberg, and Collin Winter, Tricorder: Building a Program Analysis Ecosystem, International Conference on Software Engineering (ICSE), May 2015.
>
> 7  Caitlin Sadowski, Jeffrey van Gogh, Ciera Jaspan, Emma Söderberg, and Collin Winter, Tricorder: 建立一個程式分析生態系統，國際軟體工程會議（ICSE），2015年5月。
>
> [^8]: Caitlin Sadowski, Edward Aftandilian, Alex Eagle, Liam Miller-Cushon, and Ciera Jaspan, 『Lessons from Building Static Analysis Tools at Google』, Communications of the ACM, 61 No. 4 (April 2018): 58–66, `https://cacm.acm.org/magazines/2018/4/226371-lessons-from-building-static-analysis-tools-at-google/fulltext`.
>
> 8 Caitlin Sadowski, Edward Aftandilian, Alex Eagle, Liam Miller-Cushon, and Ciera Jaspan, 『Lessons from Building Static Analysis Tools at Google』, ACM通訊期刊, 61 No. 4 (April 2018): 58–66, `https:// cacm.acm.org/magazines/2018/4/226371-lessons-from-building-static-analysis-tools-at-google/fulltext`.

### Integrated Tools  整合工具

There are many different types of static analysis tools integrated with Tricorder.

Tricorder 整合了許多不同型別的靜態分析工具。

[Error Prone](http://errorprone.info/) and [clang-tidy](https://oreil.ly/DAMiv)extend the compiler to identify AST antipatterns for Java and C++, respectively. These antipatterns could represent real bugs. For example, consider the following code snippet hashing a field f of type long:

```C++
result = 31 * result + (int) (f ^ (f >>> 32));
```

Error Prone 和 clang-tidy 擴充套件了編譯器以分別識別 Java 和 C++ 的 AST 反模式。 這些反模式可能代表真正的錯誤。例如，考慮以下程式碼片段雜湊 long 型別的欄位 f：

```C++
result = 31 * result + (int) (f ^ (f >>> 32));
```

Now consider the case in which the type of f is int. The code will still compile, but the right shift by 32 is a no-op so that f is XORed with itself and no longer affects the value produced. We fixed 31 occurrences of this bug in Google」s codebase while enabling the check as a compiler error in Error Prone. There are [many more such examples](https://errorprone.info/bugpatterns). AST antipatterns can also result in code readability improvements, such as removing a redundant call to `.get()` on a smart pointer.

現在考慮f的型別是int的情況，程式碼仍然可以編譯，但是右移32是空操作，因此 f 與自身進行異或，不再影響產生的值。我們修復了 Google 程式碼庫中出現的 31 次該錯誤，同時在 Error Prone 中將檢查作為編譯器錯誤啟用。這樣的例子還有很多。 AST 反模式還可以提高程式碼的可讀性，例如刪除對智慧指標的 `.get()` 的冗餘呼叫。

Other analyzers showcase relationships between disparate files in a corpus. The Deleted Artifact Analyzer warns if a source file is deleted that is referenced by other non-code places in the codebase (such as inside checked-in documentation). IfThisThenThat allows developers to specify that portions of two different files must be changed in tandem (and warns if they are not). Chrome」s Finch analyzer runs on configuration files for A/B experiments in Chrome, highlighting common problems including not having the right approvals to launch an experiment or crosstalk with other currently running experiments that affect the same population. The Finch analyzer makes Remote Procedure Calls (RPCs) to other services in order to provide this information.

其他分析器展示了語料庫中不同檔案之間的關係。如果刪除了程式碼庫中其他非程式碼位置（例如簽入文件中）引用的源檔案，Deleted Artifact Analyzer 會發出警告。 IfThis-ThenThat 允許開發人員指定兩個不同檔案的部分必須同時更改（如果不是，則發出警告）。 Chrome 的 Finch 分析器在 Chrome 中的 A/B 實驗的設定檔案上執行，突出顯示常見問題，包括未獲得啟動實驗的正確批准或與影響同一人群的其他當前正在執行的實驗串擾。 Finch 分析器對其他服務進行遠端過程呼叫 (RPC) 以提供此訊息。

In addition to the source code itself, some analyzers run on other artifacts produced by that source code; many projects have enabled a binary size checker that warns when changes significantly affect a binary size.

除了原始碼本身之外，一些分析器還可以在該原始碼生成的其他構件上執行；許多專案啟用了二進位大小檢查器，當更改顯著影響二進位大小時會發出警告。

Almost all analyzers are intraprocedural, meaning that the analysis results are based on code within a procedure (function). Compositional or incremental interprocedural analysis techniques are technically feasible but would require additional infrastructure investment (e.g., analyzing and storing method summaries as analyzers run).

幾乎所有分析器都是面向過程內的，這意味著分析結果基於過程（函式）內的程式碼。組合或增量過程間分析技術在技術上是可行的，但需要額外的基礎設施投資（例如，在分析器執行時分析和儲存方法摘要）。

### Integrated Feedback Channels  整合反饋渠道

As mentioned earlier, establishing a feedback loop between analysis consumers and analysis writers is critical to track and maintain developer happiness. With Tricorder, we display the option to click a 『Not useful』 button on an analysis result; this click provides the option to file a bug *directly against the analyzer writer* about why the result is not useful with information about analysis result prepopulated. Code reviewers can also ask change authors to address analysis results by clicking a 『Please fix』 button. The Tricorder team tracks analyzers with high 『Not useful』 click rates, particularly relative to how often reviewers ask to fix analysis results, and will disable analyzers if they don」t work to address problems and improve the 『not useful』 rate. Establishing and tuning this feedback loop took a lot of work, but has paid dividends many times over in improved analysis results and a better user experience (UX)— before we established clear feedback channels, many developers would just ignore analysis results they did not understand.

如上所述，建立分析者和作者之間反饋閉環對於跟蹤和維護開發人員的成幸福感很重要。Tricorder會在分析結果上顯示單擊『無用』按鈕的選項；此按鈕提供了*直接針對分析器作者*送出錯誤的選項，說明了為什麼分析結果訊息無用，程式碼審查員還可以透過單擊『請修復』按鈕要求變更作者處理分析結果。 Tricorder團隊跟蹤『無用』按鈕點選率高的分析器，特別是與審閱者要求修復分析結果的頻率有關，如果分析器不能解決問題並改進『無用』，則會停用分析器。建立和調整這個反饋閉環需要大量工作，但在改進分析結果和更好的使用者體驗 (UX) 方面已經獲得了很大的回報——在我們建立清晰的反饋渠道之前，許多開發人員會忽略他們不理解的分析結果.

And sometimes the fix is pretty simple—such as updating the text of the message an analyzer outputs! For example, we once rolled out an Error Prone check that flagged when too many arguments were being passed to a printf-like function in Guava that accepted only %s (and no other printf specifiers). The Error Prone team received weekly 『Not useful』 bug reports claiming the analysis was incorrect because the number of format specifiers matched the number of arguments—all due to users trying to pass specifiers other than %s. After the team changed the diagnostic text to state directly that the function accepts only the %s placeholder, the influx of bug reports stopped. Improving the message produced by an analysis provides an explanation of what is wrong, why, and how to fix it exactly at the point where that is most relevant and can make the difference for developers learning something when they read the message.

有時修復非常簡單，例如更新分析器輸出的訊息文字。 我們曾經推出了一個容易出錯的檢查，當太多引數被傳遞給Guava中的類似printf的函式時，該檢查只接受%s(並且不接受其他printf說明符）。Error Prone團隊每週都會收到『無用』的錯誤報告，聲稱分析不正確，因為格式說明符的數量與引數的數量相對應——所有這些都是由於使用者試圖傳遞除 %s 之外的說明符。在團隊將診斷文字更改為直接宣告該函式僅接受 %s 佔位符後，錯誤報告的湧入停止了。 改進分析產生的訊息可以解釋什麼是錯誤的、為什麼以及如何在最相關的點上準確地修復它，並且可以對開發人員在閱讀訊息時學習一些東西產生影響。

### Suggested Fixes  建議的修復

Tricorder checks also, when possible, *provide fixes*, as shown in Figure 20-2.

Tricorder 檢查也會在可能的情況下提供修復，如圖 20-2 所示。

![Figure 20-2](./images/Figure%2020-2.png)

*Figure 20-2. View of an example static analysis fix in Critique*  圖20-2. Critique中靜態分析修復的例子檢視

Automated fixes serve as an additional documentation source when the message is unclear and, as mentioned earlier, reduce the cost to addressing static analysis issues. Fixes can be applied directly from within Critique, or over an entire code change via a command-line tool. Although not all analyzers provide fixes, many do. We take the approach that *style* issues in particular should be fixed automatically; for example, by formatters that automatically reformat source code files. Google has style guides for each language that specify formatting issues; pointing out formatting errors is not a good use of a human reviewer」s time. Reviewers click 『Please Fix』 thousands of times per day, and authors apply the automated fixes approximately 3,000 times per day. And Tricorder analyzers received 『Not useful』 clicks 250 times per day.

當反饋訊息不清晰時，自動修復可作為額外的文件來源，並且可以降低解決靜態分析問題的成本。 修復可以直接應用Critique中，也可以透過指令行工具應用於整個程式碼更改。並非所有分析器都提供修復，但很多都有。 我們的做法是，優先自動修復樣式問題， 例如，透過自動重新格式化原始碼檔案的格式化程式。 Google 有每種語言的風格指南，規定了各種語言的格式，但指出格式錯誤並不能很好地利用審閱者的時間。稽核者每天點選數千次『請修復』，作者每天應用自動修復大約3,000次，Tricorder分析器每天收到250次『無用』點選

### Per-Project Customization  按專案定製

After we had built up a foundation of user trust by showing only high-confidence analysis results, we added the ability to run additional 『optional』 analyzers to specific projects in addition to the on-by-default ones. The *Proto Best Practices* analyzer is an example of an optional analyzer. This analyzer highlights potentially breaking data  format changes to [protocol buffers](https://developers.google.com/protocol-buffers)—Google」s language-independent data serialization format. These changes are only breaking when serialized data is stored somewhere (e.g., in server logs); protocol buffers for projects that do not have stored serialized data do not need to enable the check. We have also added the ability to customize existing analyzers, although typically this customization is limited, and many checks are applied by default uniformly across the codebase.

在透過僅顯示高置信度分析結果建立使用者信任基礎後，除了預設啟用的分析器之外，我們還新增了對特定專案執行其他『可選』分析器的能力。 比如Proto Best Practices 分析器，此分析器突出顯示潛在的破壞性資料協定緩衝區的格式更改——Google 的獨立於語言的資料序列化格式。只有當序列化的資料儲存在某個地方（例如，在服務器日誌中）時，這些更改才會中斷；沒有儲存序列化資料的專案的協定緩衝區不需要啟用檢查。我們還新增了自定義現有分析器的功能，儘管這種自定義功能很有限，並且預設情況下，許多檢查在程式碼庫中統一應用。

Some analyzers have even started as optional, improved based on user feedback, built up a large userbase, and then graduated into on-by-default status as soon as we could capitalize on the user trust we had built up. For example, we have an analyzer that suggests Java code readability improvements that typically do not actually change code behavior. Tricorder users initially worried about this analysis being too 『noisy,』 but eventually wanted more analysis results available.

一些分析器甚至一開始是可選的，根據使用者反饋進行改進，建立了龐大的使用者群，然後一旦我們可以利用我們建立的使用者信任，就進入預設狀態。例如，我們有一個分析器，它建議 Java 程式碼可讀性改進，這些改進通常不會真正改變程式碼行為。Tricorder使用者最初擔心這種分析過於『嘈雜』，但最終希望獲得更多的分析結果。

The key insight to making this customization successful was to focus on *project-level* *customization, not user-level customization*. Project-level customization ensures that all team members have a consistent view of analysis results for their project and prevents situations in which one developer is trying to fix an issue while another developer introduces it.

這種定製成功的關鍵是專注於專案定製，而不是使用者級定製。專案級定製確保所有團隊成員對其專案的分析結果有一致的看法，並減少一個開發人員試圖解決問題而需要另一位開發人員介紹的情況。

Early on in the development of Tricorder, a set of relatively straightforward style checkers (『linters』) displayed results in Critique, and Critique provided user settings to choose the confidence level of results to display and suppress results from specific analyses. We removed all of this user customizability from Critique and immediately started getting complaints from users about annoying analysis results. Instead of reenabling customizability, we asked users why they were annoyed and found all kinds of bugs and false positives with the linters. For example, the C++ linter also ran on Objective-C files but produced incorrect, useless results. We fixed the linting infrastructure so that this would no longer happen. The HTML linter had an extremely high false-positive rate with very little useful signal and was typically suppressed from view by developers writing HTML. Because the linter was so rarely helpful, we just disabled this linter. In short, user customization resulted in hidden bugs and suppressing feedback.

Tricorder開發的早期，Critique展示了一組相對簡單的樣式檢查器（『linter』），Critique提供了使用者設定來選擇結果的置信度以顯示和抑制來自特定分析的結果。我們從 Critique 中刪除了所有這些使用者可定製性，並立即開始收到使用者對煩人的分析結果的投訴。我們沒有重新啟用可定製性，而是詢問使用者為什麼他們感到惱火，並發現 linter 存在各種錯誤和誤報。例如，C++ linter 也在 Objective-C 檔案上執行，但產生了不正確、無用的結果。我們修復了 linting 基礎設施，這樣就不會再發生這種情況了。 HTML linter 的誤報率非常高，有用的訊號很少，並且通常被編寫 HTML 的開發人員禁止檢視。因為 linter 很少有幫助，所以我們只是停用了這個 linter。簡而言之，使用者定製導致隱藏的錯誤和抑制反饋。

### Presubmits  預送出

In addition to code review, there are also other workflow integration points for static analysis at Google. Because developers can choose to ignore static analysis warnings displayed in code review, Google additionally has the ability to add an analysis that blocks committing a pending code change, which we call a *presubmit check*. Presubmit checks include very simple customizable built-in checks on the contents or metadata of a change, such as ensuring that the commit message does not say 『DO NOT SUBMIT』 or that test files are always included with corresponding code files. Teams can also specify a suite of tests that must pass or verify that there are no Tricorder  issues for a particular category. Presubmits also check that code is well formatted. Presubmit checks are typically run when a developer mails out a change for review and again during the commit process, but they can be triggered on an ad hoc basis in between those points. See Chapter 23 for more details on presubmits at Google.

除了程式碼審查之外， Google 還有其他用於靜態分析的工作流整合點。由於開發人員可以選擇忽略程式碼審查中顯示的靜態分析警告， Google 還可以新增一個分析來阻止送出待處理的程式碼更改，我們稱之為*預送出檢查*。預送出檢查包括對更改的內容或元資料的非常簡單的可定製的內建檢查，例如確保送出訊息沒有說『不要送出』或測試檔案始終包含在相應的程式碼檔案中。團隊還可以指定一組測試，這些測試必須透過或驗證特定類別別沒有 Tricorder 問題。預送出還會檢查程式碼是否格式正確。預送出檢查通常在開發人員郵寄更改以供稽核時運行，並在送出過程中再次執行，但它們可以在這些點之間臨時觸發。有關 Google 預送出的更多詳細訊息，請參閱第 23 章。

Some teams have written their own custom presubmits. These are additional checks on top of the base presubmit set that add the ability to enforce higher best-practice standards than the company as a whole and add project-specific analysis. This enables new projects to have stricter best-practice guidelines than projects with large amounts of legacy code (for example). Team-specific presubmits can make the large- scale change (LSC) process (see Chapter 22) more difficult, so some are skipped for changes with 『CLEANUP=』 in the change description.

一些團隊已經編寫了自己的自定義預送出。這些是在基本預送出集之上的額外檢查，增加了執行比整個公司更高的最佳實踐標準的能力，並新增了特定於專案的分析。這使得新專案比擁有大量遺留程式碼的專案（例如）擁有更嚴格的最佳實踐指南。團隊特定的預送出會使大規模變更 (LSC) 過程（參見第 22 章）更加困難，因此在變更描述中帶有『CLEANUP=』的變更會被跳過。

### Compiler Integration 編譯器整合

Although blocking commits with static analysis is great, it is even better to notify developers of problems even earlier in the workflow. When possible, we try to push static analysis into the compiler. Breaking the build is a warning that is not possible to ignore, but is infeasible in many cases. However, some analyses are highly mechanical and have no effective false positives. An example is [Error Prone 『ERROR』 checks](https://errorprone.info/bugpatterns). These checks are all enabled in Google」s Java compiler, preventing instances of the error from ever being introduced again into our codebase. Compiler checks need to be fast so that they don」t slow down the build. In addition, we enforce these three criteria (similar criteria exist for the C++ compiler):

- Actionable and easy to fix (whenever possible, the error should include a suggested fix that can be applied mechanically)
- Produce no effective false positives (the analysis should never stop the build for correct code)
- Report issues affecting only correctness rather than style or best practices

儘管使用靜態分析阻止送出很好用，但最好在工作流程的早期通知開發人員問題。 如果可以的話，我們會嘗試將靜態分析推送到編譯器中。 破壞建立是一個不可忽視的警告，但在許多情況下是不可行的。然而，一些分析是高度機械化的，沒有有效的誤報。 一個例子是容易出錯的『錯誤』檢查， 這些檢查都在 Google 的 Java 編譯器中啟用，防止錯誤實例再次被引入我們的程式碼庫， 編譯器檢查需要快速，以免減慢建立速度。此外，我們強制執行這三個標準（C++ 編譯器也存在類似的標準）：

- 可操作且易於修復（只要可能，錯誤應包括可自動應用的建議修復）
- 不產生有效的誤報（分析不應停止生成正確的程式碼）
- 報告僅影響正確性而非風格或最佳實踐的問題

To enable a new check, we first need to clean up all instances of that problem in the codebase so that we don」t break the build for existing projects just because the compiler has evolved. This also implies that the value in deploying a new compiler-based check must be high enough to warrant fixing all existing instances of it. Google has infrastructure in place for running various compilers (such as clang and javac) over the entire codebase in parallel via a cluster—as a MapReduce operation. When compilers are run in this MapReduce fashion, the static analysis checks run must produce fixes in order to automate the cleanup. After a pending code change is prepared and tested that applies the fixes across the entire codebase, we commit that change and remove all existing instances of the problem. We then turn the check on in the compiler so that no new instances of the problem can be committed without breaking the build. Build breakages are caught after commit by our Continuous Integration (CI) system, or before commit by presubmit checks (see the earlier discussion).

要啟用新的檢查，我們首先需要清理程式碼庫中該問題的所有實例，這樣我們就不會因為編譯器的發展而破壞現有專案的建立。這也意味著部署新的基於編譯器的檢查的價值必須足夠高，以保證修復它的所有現有實例。Google 有基礎設施，可以透過叢集在整個程式碼庫上並行執行各種編譯器（例如 clang 和 javac）——作為 MapReduce 操作。當編譯器以這種 MapReduce 方式執行時，執行的靜態分析檢查必須產生修復以自動進行清理。在準備好並測試了在整個程式碼庫中應用修復的待處理程式碼更改後，我們送出該更改並刪除所有現有的問題實例。然後我們在編譯器中開啟檢查，這樣就不會在不破壞建立的情況下送出問題的新實例。在我們的持續整合 (CI) 系統送出之後，或者在送出之前透過預送出檢查（參見前面的討論）捕獲建立損壞。

We also aim to never issue compiler warnings. We have found repeatedly that developers ignore compiler warnings. We either enable a compiler check as an error (and break the build) or don」t show it in compiler output. Because the same compiler flags are used throughout the codebase, this decision is made globally. Checks that can」t be made to break the build are either suppressed or shown in code review (e.g., through Tricorder). Although not every language at Google has this policy, the most frequently used ones do. Both of the Java and C++ compilers have been configured to avoid displaying compiler warnings. The Go compiler takes this to extreme; some things that other languages would consider warnings (such as unused variables or package imports) are errors in Go.

我們的目標是永遠不會發出編譯器警告，但是我們不斷的發現開發人員會忽略編譯器警告，要麼啟用編譯器檢查作為錯誤（並中斷建立），要麼不在編譯器輸出中顯示它。因為在整個程式碼庫中使用相同的編譯器標誌，所以這個決定是全域做出的。無法破壞建立的檢查要麼被抑制，要麼在程式碼審查中顯示（例如，透過 Tricorder）。儘管並非 Google 的所有語言都有此策略，但最常用的語言都有。Java 和 C++ 編譯器都已設定為避免顯示編譯器警告，Go 編譯器將這一點做的很好，因為在其他語言中會考慮警告的一些事情（例如未使用的變數或套件匯入），在 Go 中是錯誤的。

### Analysis While Editing and Browsing Code  編輯和瀏覽程式碼時分析

Another potential integration point for static analysis is in an integrated development environment (IDE). However, IDE analyses require quick analysis times (typically less than 1 second and ideally less than 100 ms), and so some tools are not suitable to integrate here. In addition, there is the problem of making sure the same analysis runs identically in multiple IDEs. We also note that IDEs can rise and fall in popularity (we don」t mandate a single IDE); hence IDE integration tends to be messier than plugging into the review process. Code review also has specific benefits for displaying analysis results. Analyses can take into account the entire context of the change; some analyses can be inaccurate on partial code (such as a dead code analysis when a function is implemented before adding callsites). Showing analysis results in code review also means that code authors have to convince reviewers as well if they want to ignore analysis results. That said, IDE integration for suitable analyses is another great place to display static analysis results.

靜態分析的另一個整合點是整合開發環境 (IDE)。但是，IDE 分析需要快速的分析時間（通常小於 1 秒，理想情況下小於 100 毫秒），因此某些工具不適合在這裡整合，此外，還存在確保相同分析在多個 IDE 中以相同方式執行的問題。我們還發現 IDE 的受歡迎程度可能會上升或下降（我們不強制要求單一的 IDE），因此 IDE 整合往往比插入審查過程更混亂。程式碼審查還具有顯示分析結果的特定好處。分析可以考慮變更的整個背景，某些對部分程式碼點分析可能不準確（例如，在新增呼叫點之前實現函式時的死程式碼分析）。在程式碼審查中顯示分析結果也意味著如果程式碼作者想忽略分析結果，他們也必須透過審查。也就是說，IDE整合進行適當的分析是顯示靜態分析結果的一個不錯的整合點。

Although we mostly focus on showing newly introduced static analysis warnings, or warnings on edited code, for some analyses, developers actually do want the ability to view analysis results over the entire codebase during code browsing. An example of this are some security analyses. Specific security teams at Google want to see a holistic view of all instances of a problem. Developers also like viewing analysis results over the codebase when planning a cleanup. In other words, there are times when showing results when code browsing is the right choice.

儘管我們主要關注顯示新引入的靜態分析警告或編輯程式碼的警告，但對於某些分析，開發人員實際上確實希望能夠在程式碼瀏覽期間檢視整個程式碼庫的分析結果。這方面的例子是一些安全分析。 Google 的特定安全團隊希望檢視所有問題實例的整體檢視。開發人員還喜歡在計劃清理時透過程式碼庫檢視分析結果。換句話說，在瀏覽程式碼的時候顯示結果是正確的選擇。

## Conclusion  總結

Static analysis can be a great tool to improve a codebase, find bugs early, and allow more expensive processes (such as human review and testing) to focus on issues that are not mechanically verifiable. By improving the scalability and usability of our static analysis infrastructure, we have made static analysis an effective component of software development at Google.

靜態分析是一個很好的工具，可以改進程式碼庫，儘早發現錯誤，並允許成本更高的過程（如人工審查和測試）聚焦在無法透過機器方式驗證的問題。透過提高靜態分析基礎設施的可擴充性和可用性，我們使靜態分析成為 Google 軟體開發的有效組成部分。

## 內容提要

- *Focus on developer happiness*. We have invested considerable effort in building feedback channels between analysis users and analysis writers in our tools, and aggressively tune analyses to reduce the number of false positives.
- *Make static analysis part of the core developer workflow*. The main integration point for static analysis at Google is through code review, where analysis tools provide fixes and involve reviewers. However, we also integrate analyses at additional points (via compiler checks, gating code commits, in IDEs, and when browsing code).
- *Empower users to contribute*. We can scale the work we do building and maintaining analysis tools and platforms by leveraging the expertise of domain experts. Developers are continuously adding new analyses and checks that make their lives easier and our codebase better.

- **關注開發者的幸福感**。我們投入了大量精力，在我們的工具中建立分析使用者和作者之間的反饋渠道，並積極調整分析以減少誤報的數量。
- **將靜態分析作為核心開發人員工作流程的一部分**。 Google 靜態分析的主要整合點是透過程式碼評審，在這裡，分析工具提供修復並讓評審人員參與。然而，我們也在其他方面（透過編譯器檢查、選通程式碼送出、在IDE中以及在瀏覽程式碼時）整合分析。
- **授權使用者做出貢獻**。透過利用領域專家的專業知識，我們可以擴充套件建立和維護分析工具和平台的工作。開發人員不斷新增新的分析和檢查，使他們的生活更輕鬆，使我們的程式碼庫更好。
