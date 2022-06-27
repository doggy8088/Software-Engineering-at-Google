**CHAPTER 19**

# Critique: Google’s Code Review Tool

# 第十九章 體驗：google的程式碼審查工具

            Written by  Written by Caitlin Sadowski, Ilham Kurnia, and Ben Rohlfs Edited by Lisa Carey

As you saw in Chapter 9, code review is a vital part of software development, particularly when working at scale. The main goal of code review is to improve the readability and maintainability of the code base, and this is supported fundamentally by the review process. However, having a well-defined code review process in only one part of the code review story. Tooling that supports that process also plays an important part in its success.

正如你在第9章中所看到的，程式碼審查是軟體開發的重要組成部分，特別是在大規模工作時。程式碼審查的主要目標是提高程式碼函式庫的可讀性和可維護性，評審過程從根本上支援這一點。然而，擁有一個定義明確的程式碼審查過程只是程式碼審查流程的一個部分。支援該過程的工具在其成功中也起著重要作用。

In this chapter, we’ll look at what makes successful code review tooling via Google’s well-loved in-house system, Critique. Critique has explicit support for the primary motivations of code review, providing reviewers and authors with a view of the review and ability to comment on the change. Critique also has support for gatekeeping what code is checked into the codebase, discussed in the section on “scoring” changes. Code review information from Critique also can be useful when doing code archaeology, following some technical decisions that are explained in code review interactions (e.g., when inline comments are lacking). Although Critique is not the only code review tool used at Google, it is the most popular one by a large margin.

在本章中，我們將透過Google深受喜愛的內部系統Critique，來看看成功的程式碼審查工具的模樣。Critique明確支援程式碼審查的主要功能，為審查者和作者提供審查的檢視和對修改的評論能力。Critique還支援對哪些程式碼被檢入程式碼函式庫進行把關，這一點在 "評分 "更改一節中討論。評論中的程式碼評審資訊在進行程式碼考古時也很有用，遵循程式碼評審互動中解釋的一些技術決策（例如，當缺少內聯註釋時）。儘管Critique並不是Google唯一使用的程式碼審查工具，但它是最受歡迎的工具。

## Code Review Tooling Principles 程式碼審查工具原則

We mentioned above that Critique provides functionality to support the goals of code review (we look at this functionality in more detail later in this chapter), but why is it so successful? Critique has been shaped by Google’s development culture, which includes code review as a core part of the workflow. This cultural influence translates into a set of guiding principles that Critique was designed to emphasize:
*Simplicity*
	Critique’s user interface (UI) is based around making it easy to do code review without a lot of unnecessary choices, and with a smooth interface. The UI loads fast, navigation is easy and hotkey supported, and there are clear visual markers for the overall state of whether a change has been reviewed.
*Foundation of trust*
	Code review is not for slowing others down; instead, it is for empowering others. Trusting colleagues as much as possible makes it work. This might mean, for example, trusting authors to make changes and not requiring an additional review phase to double check that minor comments are actually addressed. Trust also plays out by making changes openly accessible (for viewing and reviewing) across Google.
*Generic communication*
	Communication problems are rarely solved through tooling. Critique prioritizes generic ways for users to comment on the code changes, instead of complicated protocols. Critique encourages users to spell out what they want in their comments or even suggests some edits instead of making the data model and process more complex. Communication can go wrong even with the best code review tool because the users are humans.
*Workflow integration*
	Critique has a number of integration points with other core software development tools. Developers can easily navigate to view the code under review in our code search and browsing tool, edit code in our web-based code editing tool, or view test results associated with a code change.

我們在前面提到，Critique提供了支援程式碼審查目標的功能（我們在本章後面會詳細介紹這種功能），但為什麼它如此成功？Critique是基於Google的開發文化塑造的，其中包括程式碼審查作為工作流程的核心部分。這種文化影響轉化為一套指導原則，Critique的設計就是為了強調這些原則：
*簡潔性
	Critique的使用者介面（UI）基於使程式碼審查變得容易而不需要很多不必要的選擇，並且具有流暢介面。使用者介面載入速度快，導航簡單，支援熱鍵，而且有清晰的視覺標記，可以顯示更改是否已稽核的總體狀態。
*信任的基礎*
	程式碼審查不是為了拖慢別人，相反，它是為了授權他人。儘可能地信任同事使其發揮作用。這可能意味著，例如，信任作者進行修改，而不需要額外的審查階段來再次檢查是否確實解決了次要評論。信任還體現在使修改在整個 Google 上公開進行（供檢視和審查）。
*通用的溝通*
	溝通問題很難透過工具來解決。Critique優先考慮讓使用者對程式碼修改進行評論的通用方法，而不是複雜的協定。評論鼓勵使用者詳細說明他們想要的內容，甚至建議進行一些編輯，而不是使資料模型和過程更加複雜。即使是最好的程式碼審查工具，溝通也會出錯，因為使用者是人。
*工作流程的整合*。
	Critique有很多與其他核心軟體開發工具的整合點。開發人員可以在我們的程式碼搜尋和瀏覽工具中輕鬆瀏覽正在審查的程式碼，在我們基於網路的程式碼編輯工具中編輯程式碼，或者檢視與程式碼修改相關的測試結果。

Across these guiding principles, simplicity has probably had the most impact on the tool. There were many interesting features we considered adding, but we decided not to make the model more complicated to support a small set of users.

在這些指導原則中，簡單性可能對這個工具影響最大。我們考慮過增加許多有趣的功能，但我們決定不為支援一小部分使用者而使模型更加複雜。

Simplicity also has an interesting tension with workflow integration. We considered but ultimately decided against creating a “Code Central” tool with code editing, reviewing, and searching in one tool. Although Critique has many touchpoints with other tools, we consciously decided to keep code review as the primary focus. Features are linked from Critique but implemented in different subsystems.

簡單與工作流程的整合也有一個有趣的矛盾。我們考慮過，但最終決定不建立一個集程式碼編輯、審查和搜尋於一體的 "程式碼中心 "工具。儘管Critique與其他工具有許多接觸點，但我們還是有意識地決定將程式碼審查作為主要關注點。特徵與評論相關，但在不同的子系統中實施。

## Code Review Flow 程式碼審查流程
Code reviews can be executed at many stages of software development, as illustrated in Figure 19-1. Critique reviews typically take place before a change can be committed to the codebase, also known as precommit reviews. Although Chapter 9 contains a brief description of the code review flow, here we expand it to describe key aspects of Critique that help at each stage. We’ll look at each stage in more detail in the following sections.

程式碼審查可以在軟體開發的許多階段進行，如圖19-1所示。評論評審通常在變更提交到程式碼函式庫之前進行，也稱為預提交評審。儘管第9章包含了對程式碼評審流程的簡要描述，但在這裡我們將其擴充套件，以描述Critique在每個階段的關鍵作用。我們將在下面的章節中更詳細地討論每個階段。

![Figure 19-1](./images/Figure%2019-1.png)

*Figure 19-1. The code review flow*

Typical review steps go as follows:

1. **Create a change.** A user authors a change to the codebase in their workspace. This *author* then uploads a *snapshot* (showing a patch at a particular point in time) to Critique, which triggers the run of automatic code analyzers (see [Chapter 20](#_bookmark1781)).
2. **Request** **review.** After the author is satisfied with the diff of the change and the result of the analyzers shown in Critique, they mail the change to one or more reviewers.
3. **Comment.** *Reviewers* open the change in Critique and draft comments on the diff. Comments are by default marked as *unresolved,* meaning they are crucial for the author to address. Additionally, reviewers can add *resolved* comments that are optional or informational. Results from automatic code analyzers, if present, are also visible to reviewers. Once a reviewer has drafted a set of comments, they need to *publish* them in order for the author to see them; this has the advantage of allowing a reviewer to provide a complete thought on a change atomically, after having reviewed the entire change. Anyone can comment on changes, providing a “drive-by review” as they see it necessary.
4. **Modify change and reply to comments.** The author modifies the change, uploads new snapshots based on the feedback, and replies back to the reviewers. The author addresses (at least) all unresolved comments, either by changing the code or just replying to the comment and changing the comment type to be *resolved*. The author and reviewers can look at diffs between any pairs of snapshots to see what changed. Steps 3 and 4 might be repeated multiple times.
5. **Change approval.** When the reviewers are happy with the latest state of the change, they approve the change and mark it as “looks good to me” (LGTM). They can optionally include comments to address. After a change is deemed good for submission, it is clearly marked green in the UI to show this state.
6. **Commit a change.** Provided the change is approved (which we’ll discuss shortly), the author can trigger the commit process of the change. If automatic analyzers and other precommit hooks (called “presubmits”) don’t find any problems, the change is committed to the codebase.

典型的審查步驟如下：
1. **建立一個變更。**一個使用者對其工作區的程式碼函式庫進行變更。然後這個*作者*向Critique上傳一個*快照*（顯示某一特定時間點的補丁），這將觸發自動程式碼分析器的執行（見[第20章](#_bookmark1781)）。
2. **要求審查。**在作者對修改的差異和Critique中顯示的分析器的結果感到滿意後，他們將修改傳送給一個或多個審查員。
3. **評論。**評論者在Critique中開啟修改，並對diff起草評論。評論預設標記為*未解決*，意味著它們對作者來說是至關重要的。此外，評論者可以新增*已解決*的評論，這些評論是可選的或資訊性的。自動程式碼分析器的結果，如果存在的話，也可以讓審查者看到。一旦審查者起草了一組評論，他們需要*發佈*它們，以便作者看到它們；這樣做的好處是允許審查者在審查了整個修改後，以原子方式提供一個完整的想法。任何人都可以對變更發表評論，並在他們認為必要時提供“驅動式審查”。
4. **修改變更並回複評論。**作者修改變更，根據反饋上傳新的快照，並回複評論者。作者處理（至少）所有未解決的評論，要麼修改程式碼，要麼直接回複評論並將評論型別改為*解決*。作者和審稿人可以檢視任何一對快照之間的差異，看看有什麼變化。步驟3和4可能要重複多次。
5. **變更批准。**當審查者對修改的最新狀態感到滿意時，他們會批准變更，並將其標記為 “我覺得不錯"（LGTM）。他們可以選擇包括評論來解決。更改被認為適合提交後，在UI中會清楚地標記為綠色以顯示此狀態。
6. **提交變更。**只要變更被批准（我們很快會討論），作者就可以觸發變更的提交過程。如果自動分析器和其他預提交鉤子（稱為 "預提交"）沒有發現任何問題，該變更就被提交到程式碼函式庫中。

Even after the review process is started, the entire system provides significant flexibility to deviate from the regular review flow. For example, reviewers can un-assign themselves from the change or explicitly assign it to someone else, and the author can postpone the review altogether. In emergency cases, the author can forcefully commit their change and have it reviewed after commit.

即使在審查過程開始後，整個系統也提供了很大的靈活性來偏離常規的審查流程。例如，評審員可以取消自己對修改的分配，或者明確地將其分配給其他人，而作者可以完全推遲評審。在緊急情況下，作者可以強行提交他們的修改，並在提交後對其進行審查。

### Notifications 通知

As a change moves through the stages outlined earlier, Critique publishes event notifications that might be used by other supporting tools. This notification model allows Critique to focus on being a primary code review tool instead of a general purpose tool, while still being integrated into the developer workflow. Notifications enable a separation of concerns such that Critique can just emit events and other systems build off of those events.

當一個變更經過前面概述的階段時，Critique 會發布可能被其他支援工具使用的事件通知。這種通知模式使Critique能夠專注於成為一個主要的程式碼審查工具，而不是一個通用的工具，同時仍然能夠整合到開發人員的工作流程中。通知實現了關注點的分離，這樣Critique就可以直接發出事件，而其他系統則基於這些事件進行開發。

For example, users can install a Chrome extension that consumes these event notifications. When a change needs the user’s attention—for example, because it is their turn to review the change or some presubmit fails—the extension displays a Chrome notification with a button to go directly to the change or silence the notification. We have found that some developers really like immediate notification of change updates, but others choose not to use this extension because they find it is too disruptive to their flow.

例如，使用者可以安裝使用這些事件通知的Chrome擴充套件。當一個變更需要使用者注意時--例如，當更改需要使用者注意時，由於輪到使用者檢視更改或某個預提交失敗--該擴充套件會顯示一個Chrome通知，其中有一個按鈕可直接轉到更改或使通知靜音。我們發現，一些開發者非常喜歡即時的變更更新通知，但也有人選擇不使用這個擴充套件，因為他們覺得這對他們的工作流程太過干擾。

Critique also manages emails related to a change; important Critique events trigger email notifications. In addition to being displayed in the Critique UI, some analyzer findings are configured to also send the results out by email. Critique also processes email replies and translates them to comments, supporting users who prefer an email-based flow. Note that for many users, emails are not a key feature of code review; they use Critique’s dashboard view (discussed later) to manage reviews.

Critique還管理與變化有關的電子郵件；重要的Critique事件會觸發電子郵件通知。除了在 Critique UI 中顯示外，一些分析器的結果也被配置為透過電子郵件傳送。Critique 還處理電子郵件回覆並將其轉換為評論，支援喜歡基於電子郵件的流程的使用者。請注意，對許多使用者來說，電子郵件並不是程式碼審查的一個關鍵特徵；他們使用 Critique 的儀表盤檢視（後面會討論）來管理評論。

## Stage 1: Create a Change 階段1：建立一個變更

A code review tool should provide support at all stages of the review process and should not be the bottleneck for committing changes. In the prereview step, making it easier for change authors to polish a change before sending it out for review helps reduce the time taken by the reviewers to inspect the change. Critique displays change diffs with knobs to ignore whitespace changes and highlight move-only changes. Critique also surfaces the results from builds, tests, and static analyzers, including style checks (as discussed in [Chapter 9](#_bookmark664)).

程式碼審查工具應該在審查過程的各個階段提供支援，不應該成為提交更改的瓶頸。在審查前的步驟中，讓修改者在送出審查前更容易打磨修正，有助於減少審查者檢查修改的時間。Critique在顯示修改差異時，可以忽略空白處的修改，並突出顯示純移動的修改。Critique還可以顯示建構、測試和靜態分析器的結果，包括樣式檢查（如第9章中所討論的）。

Showing an author the diff of a change gives them the opportunity to wear a different hat: that of a code reviewer. Critique lets a change author see the diff of their changes as their reviewer will, and also see the automatic analysis results. Critique also supports making lightweight modifications to the change from within the review tool and suggests appropriate reviewers. When sending out the request, the author can also include preliminary comments on the change, providing the opportunity to ask reviewers directly about any open questions. Giving authors the chance to see a change just as their reviewers do prevents misunderstanding.

向作者展示修改的差異，讓他們有機會擁有不同的思路：程式碼審查者的思路。Critique可以讓修改作者像他們的審查者一樣看到他們的修改的差異，也可以看到自動分析的結果。Critique還支援在審查工具中對變更進行輕量級的修改，並推薦合適的審查者。在傳送請求時，作者也可以包括對修改的初步評論，提供機會直接向審查者詢問任何公開的問題。讓作者有機會像他們的審查者一樣看到一個變化，可以防止誤解。

To provide further context for the reviewers, the author can also link the change to a specific bug. Critique uses an autocomplete service to show relevant bugs, prioritizing bugs that are assigned to the author.

為了給審閱者提供進一步的上下文，作者還可以將更改連結到特定的bug。評論使用自動完成服務來顯示相關的bug，並對分配給作者的bug進行優先順序排序。

### Diffing 差異點
The core of the code review process is understanding the code change itself. Larger changes are typically more difficult to understand than smaller ones. Optimizing the diff of a change is thus a core requirement for a good code review tool.

程式碼審查過程的核心是理解程式碼變更本身。較大的變化通常比小的變化更難理解。因此，優化變更的差異是一個好的程式碼審查工具的核心要求。

In Critique, this principle translates onto multiple layers (see Figure 19-2). The diffing component, starting from an optimized longest common subsequence algorithm, is enhanced with the following:
•	Syntax highlighting
•	Cross-references (powered by Kythe; see Chapter 17)
•	Intraline diffing that shows the difference on character-level factoring in the word boundaries (Figure 19-2)
•	An option to ignore whitespace differences to a varying degree
•	Move detection, in which chunks of code that are moved from one place to another are marked as being moved (as opposed to being marked as removed here and added there, as a naive diff algorithm would)

在Critique中，這一原則轉化為多個層面（見圖19-2）。從優化的最長共同子序列演算法開始，diffing元件得到了以下增強：
- 語法高亮
- 交叉參考（由Kythe提供，見第17章）
- 字元內差分，顯示字元級的差異，並考慮到詞的邊界（圖19-2）。
- 在不同程度上忽略空白差異的選項。
- 移動檢測，在這種檢測中，從一個地方移動到另一個地方的程式碼塊被標記為正在移動（而不是像樸素的diff演算法那樣，在這裡被標記為刪除，在那裡被新增）。

![Figure 19-2](./images/Figure%2019-2.png)

*Figure 19-2. Intraline diffing showing character-level differences*

Users can also view the diff in various different modes, such as overlay and side by side. When developing Critique, we decided that it was important to have side-by- side diffs to make the review process easier. Side-by-side diffs take a lot of space: to make them a reality, we had to simplify the diff view structure, so there is no border, no padding—just the diff and line numbers. We also had to play around with a variety of fonts and sizes until we had a diff view that accommodates even for Java’s 100- character line limit for the typical screen-width resolution when Critique launched (1,440 pixels).

使用者還可以以各種不同的模式檢視diff，如疊加和並排。在開發Critique時，我們決定必須有並排的diff，使審查過程更容易。並排diff需要很大的空間：為了使它們成為現實，我們必須簡化diff檢視結構，因此沒有邊框，沒有填充，只有diff和行號。我們還不得不使用各種字型和尺寸，直到我們有了一種差異檢視，即使是在Critique啟動時典型的螢幕寬度解析度（1440畫素）下，也能滿足Java的100個字元行數限制。

Critique further supports a variety of custom tools that provide diffs of artifacts produced by a change, such as a screenshot diff of the UI modified by a change or configuration files generated by a change.

Critique還支援各種訂製工具，這些工具提供由變更產生的構件diff，例如由變更修改的UI螢幕截圖差異或由變更產生的配置檔案。

To make the process of navigating diffs smooth, we were careful not to waste space and spent significant effort ensuring that diffs load quickly, even for images and large files and/or changes. We also provide keyboard shortcuts to quickly navigate through files while visiting only modified sections.

為了使瀏覽diff的過程順利進行，我們小心翼翼地避免浪費空間，並花費大量精力確保diff載入迅速，即使是圖片和大檔案和/或修改。我們還提供快捷鍵，以便在僅訪問修改的部分時快速瀏覽檔案。

When users drill down to the file level, Critique provides a UI widget with a compact display of the chain of snapshot versions of a file; users can drag and drop to select which versions to compare. This widget automatically collapses similar snapshots, drawing focus to important snapshots. It helps the user understand the evolution of a file within a change; for example, which snapshots have test coverage, have already been reviewed, or have comments. To address concerns of scale, Critique prefetches everything, so loading different snapshots is very quick.

當用戶深入到檔案層面時，Critique提供了一個UI小工具，緊湊地顯示了檔案的快照版本鏈；使用者可以透過拖放來選擇要比較的版本。這個小元件會自動摺疊相似的快照，將注意力集中在重要的快照上。它幫助使用者理解檔案在變更中的演變；例如，哪些快照有測試覆蓋率，已經被審查過，或者有評論。為了解決規模問題，Critique預取了所有內容，所以載入不同的快照非常快。

### Analysis Results 分析結果

Uploading a snapshot of the change triggers code analyzers (see [Chapter 20](#_bookmark1781)). Critique displays the analysis results on the change page, summarized by analyzer status chips shown below the change description, as depicted in [Figure 19-3](#_bookmark1742), and detailed in the Analysis tab, as illustrated in [Figure 19-4](#_bookmark1743).

上傳變更的快照會觸發程式碼分析器（見第20章）。Critique將分析結果顯示在變更頁面上，按分析器狀態籌碼彙總，顯示在變更描述下面，如圖19-3所示，並在分析標籤中詳細說明，如圖19-4所示。

Analyzers can mark specific findings to highlight in red for increased visibility. Analyzers that are still in progress are represented by yellow chips, and gray chips are displayed otherwise. For the sake of simplicity, Critique offers no other options to mark or highlight findings—actionability is a binary option. If an analyzer produces some results (“findings”), clicking the chip opens up the findings. Like comments, findings can be displayed inside the diff but styled differently to make them easily distinguishable. Sometimes, the findings also include fix suggestions, which the author can preview and choose to apply from Critique.

分析器可以標記特定的結果，以紅色突出顯示，以提高可視性。仍在進行中的分析器由黃色卡片表示，否則顯示灰色卡片。為了簡單起見，Critique沒有提供其他選項來標記或突出研究結果--可操作性是一個二元選項。如果一個分析器產生了一些結果（"研究結果"），點選卡片就可以開啟研究結果。像評論一樣，研究結果可以顯示在diff裡面，但風格不同，使它們容易區分。有時，研究結果也包括修正建議，作者可以預先檢視這些建議，並從評論中選擇應用。

![Figure 19-3](./images/Figure%2019-3.png)

*Figure* *19-3.* *Change* *summary* *and* *diff*  view

![Figure 19-4](./images/Figure%2019-4.png)

*Figure* *19-4.* *Analysis* *results*

For example, suppose that a linter finds a style violation of extra spaces at the end of the line. The change page will display a chip for that linter. From the chip, the author can quickly go to the diff showing the offending code to understand the style violation with two clicks. Most linter violations also include fix suggestions. With a click, the author can preview the fix suggestion (for example, remove the extra spaces), and with another click, apply the fix on the change.

例如，假設一個人發現行末有多餘的空格，是違反風格的。更改頁面將顯示該linter的卡片。從卡片中，作者可以快速轉到顯示違規程式碼的diff，只需點選兩次就能瞭解樣式違規。大多數違規的linter也包括修復建議。透過點選，作者可以預覽修正建議（例如，刪除多餘的空格），並透過另一次點選，在修改中應用修正。

### Tight Tool Integration 緊密的工具整合

Google has tools built on top of Piper, its monolithic source code repository (see [Chapter 16](#_bookmark1364)), such as the following:
•   Cider, an online IDE for editing source code stored in the cloud
•   Code Search, a tool for searching code in the codebase
•   Tricorder, a tool for displaying static analysis results (mentioned earlier)
•   Rapid, a release tool that packages and deploys binaries containing a series of changes
•   Zapfhahn, a test coverage calculation tool

 Google 擁有建立在Piper--其單體原始碼函式庫（見第16章）之上的工具，例如以下這些。
- Cider，用於編輯雲中儲存的原始碼的線上IDE
- 程式碼搜尋，用於在程式碼函式庫中搜索程式碼的工具
- Tricorder，用於顯示靜態分析結果的工具（前面提到）
- Rapid，個打套件和部署包含一系列更改的二進位制檔案的發佈工具
- Zapfhahn，一個測試覆蓋率計算工具

Additionally, there are services that provide context on change metadata (for example, about users involved in a change or linked bugs). Critique is a natural melting pot for a quick one-click/hover access or even embedded UI support to these systems, although we need to be careful not to sacrifice simplicity. For example, from a change page in Critique, the author needs to click only once to start editing the change further in Cider. There is support to navigate between cross-references using Kythe or view the mainline state of the code in Code Search (see [Chapter 17](#_bookmark1485)). Critique links out to the release tool so that users can see whether a submitted change is in a specific release. For these tools, Critique favors links rather than embedding so as not to distract from the core review experience. One exception here is test coverage: the information of whether a line of code is covered by a test is shown by different background colors on the line gutter in the file’s diff view (not all projects use this coverage tool).

此外，還有一些服務可以提供變更元資料的上下文（例如，關於參與變更的使用者或連結的錯誤）。Critique是一個很自然的熔爐，它可以快速地一鍵/懸停訪問這些系統，甚至支援嵌入式UI，儘管我們需要小心不要犧牲簡單性。例如，在Critique的修改頁面上，作者只需要點選一次就可以在Cider中進一步編輯修改。我們支援使用Kythe在交叉參考之間進行導航，或在程式碼搜尋中檢視程式碼的主線狀態（見第17章）。Critique連結到發佈工具，這樣使用者就可以看到提交的變更是否在一個特定的版本中。對於這些工具，Critique更傾向於連結而不是嵌入，這樣就不會分散對核心評審經驗的注意力。這裡的一個例外是測試覆蓋率：測試是否覆蓋程式碼行的資訊由檔案的diff檢視中的行槽上的不同背景色顯示（並非所有專案都使用此覆蓋率工具）。

Note that tight integration between Critique and a developer’s workspace is possible because of the fact that workspaces are stored in a FUSE-based filesystem, accessible beyond a particular developer’s computer. The Source of Truth is hosted in the cloud and accessible to all of these tools.

請注意，Critique和開發者的工作空間之間的緊密結合是可能的，因為工作空間儲存在一個基於FUSE的檔案系統中，可以在特定開發者的計算機之外訪問。真相之源託管在雲中，所有這些工具都可以訪問。

## Stage 2: Request Review 階段2：傳送審查

After the author is happy with the state of the change, they can send it for review, as depicted in [Figure 19-5](#_bookmark1751). This requires the author to pick the reviewers. Within a small team, finding a reviewer might seem simple, but even there it is useful to distribute reviews evenly across team members and consider situations like who is on vacation. To address this, teams can provide an email alias for incoming code reviews. The alias is used by a tool called *GwsQ* (named after the initial team that used this technique:  
(Google Web Server) that assigns specific reviewers based on the configuration linked to the alias. For example, a change author can assign a review to some-team-list-alias, and GwsQ will pick a specific member of some-team-list-alias to perform the review.

在作者對更改的狀態感到滿意後，他們可以把它送去審查，如圖19-5中所描述的。這需要作者挑選審查者。在一個小團隊內，尋找審查者可能看起來很簡單，但是即使在團隊成員之間均勻地分配評論，也需要考慮像是誰休假的情況。為了解決這個問題，團隊可以為收到的程式碼審查提供一個電子郵件別名。這個別名被一個叫做*GwsQ*的工具所使用（以最初使用這種技術的團隊命名：
（ Google 網路伺服器），它根據連結到別名的配置分配特定的審閱者。例如，變更作者可以將評審分配給某個團隊列表別名，GwsQ將選擇某個團隊列表別名的特定成員來執行評審。

![Figure 19-5](./images/Figure%2019-5.png)

Figure 19-5. Requesting reviewers

Given the size of Google’s codebase and the number of people modifying it, it can be difficult to find out who is best qualified to review a change outside your own project. Finding reviewers is a problem to consider when reaching a certain scale. Critique must deal with scale. Critique offers the functionality to propose sets of reviewers that are sufficient to approve the change. The reviewer selection utility takes into account the following factors:
•   Who owns the code that is being changed (see the next section)
•   Who is most familiar with the code (i.e., who recently changed it)
•   Who is available for review (i.e., not out of office and preferably in the same time zone)
•   The GwsQ team alias setup

考慮到 Google 程式碼函式庫的規模和修改程式碼的人數，很難找出誰最有資格審查你自己專案之外的變更。發現審查者在達到一定的規模時要考慮的問題。評論必須處理規模問題。Critique提供了建議一組足以批准更改的審閱者的功能。評審員的選擇工具考慮到了以下因素。
- 誰擁有被修改的程式碼（見下一節）
- 誰對該程式碼最熟悉（即，誰最近修改過該程式碼）。
- 誰可以進行審查（即沒有離開辦公室，最好是在同一時區）。
- GwsQ團隊的別名設定

Assigning a reviewer to a change triggers a review request. This request runs “presubmits” or precommit hooks applicable to the change; teams can configure the presubmits related to their projects in many ways. The most common hooks include the following:
•   Automatically adding email lists to changes to raise awareness and transparency
•   Running automated test suites for the project
•   Enforcing project-specific invariants on both code (to enforce local code style restrictions) and change descriptions (to allow generation of release notes or other forms of tracking)

為一個變更指定一個審查員會觸發一個審查請求。該請求執行適用於該變更的 "預提交 "或預提交鉤子；團隊可以以多種方式配置與他們的專案相關的預提交。最常見的鉤子包括以下內容：
- 自動將電子郵件列表新增到更改中，以提高意識和透明度
- 為專案執行自動化測試套件
- 對程式碼（強制執行原生代碼風格限制）和變更描述（允許產生發佈說明或其他形式的追蹤）執行專案特定的不變因素

As running tests is resource intensive, at Google they are part of presubmits (run when requesting review and when committing changes) rather than for every snapshot like Tricorder checks. Critique surfaces the result of running the hooks in a similar way to how analyzer results are displayed, with an extra distinction to highlight the fact that a failed result blocks the change from being sent for review or committed. Critique notifies the author via email if presubmits fail.

 由於執行測試是資源密集型的，在Google，它們是預提交的一部分（在請求審查和提交修改時執行），而不是像Tricorder檢查那樣為每個快照執行。Critique以類似於分析器結果的方式顯示執行鉤子的結果，並有一個額外的區別，即失敗的結果會阻止修改被送審或提交。如果預提交失敗，Critique會透過電子郵件通知作者。

## Stages 3 and 4: Understanding and Commenting on a Change 階段3和4：理解和評論變更

After the review process starts, the author and the reviewers work in tandem to reach the goal of committing changes of high quality.

審查過程開始後，作者和審查員協同工作，以達到提交高品質變更的目標。

### Commenting 評論

Making comments is the second most common action that users make in Critique after viewing changes (Figure 19-6). Commenting in Critique is free for all. Anyone—not only the change author and the assigned reviewers—can comment on a change.

發表評論是使用者在Critique檢視修改後的第二常見的行為（圖19-6）。評論中的評論對所有人都是公開的。任何人--不僅僅是修改作者和指定的評審者--都可以對修改進行評論。

Critique also offers the ability to track review progress via per-person state. Reviewers have checkboxes to mark individual files at the latest snapshot as reviewed, helping the reviewer keep track of what they have already looked at. When the author modifies a file, the “reviewed” checkbox for that file is cleared for all reviewers because the latest snapshot has been updated.

評論還提供了透過個人狀態追蹤審查進度的能力。審閱者有複選框將最新快照中的單個檔案標記為已審閱，以幫助審閱者追蹤他們已檢視的內容。當作者修改檔案時，所有審閱者都會清除該檔案的“審閱”複選框，因為最新快照已更新。

![Figure 19-6](./images/Figure%2019-6.png)

*Figure 19-6. Commenting on the diff view*

When a reviewer sees a relevant analyzer finding, they can click a “Please fix” button to create an unresolved comment asking the author to address the finding. Reviewers can also suggest a fix to a change by inline editing the latest version of the file. Critique transforms this suggestion into a comment with a fix attached that can be applied by the author.

當審查者看到一個相關的分析器發現時，他們可以點選 "請修復 "按鈕，建立一個未解決的評論，要求作者解決這個問題。審查者還可以透過內聯編輯檔案的最新版本來建議修改。Critique將此建議轉換為評論，並附上一個作者可以應用的修復程式。

Critique does not dictate what comments users should create, but for some common comments, Critique provides quick shortcuts. The change author can click the “Done” button on the comment panel to indicate when a reviewer’s comment has been addressed, or the “Ack” button to acknowledge that the comment has been read, typically used for informational or optional comments. Both have the effect of resolving the comment thread if it is unresolved. These shortcuts simplify the workflow and reduce the time needed to respond to review comments.

Critique 沒有規定使用者應該建立什麼評論，但對於一些常見的評論，Critique 提供了快速的快捷方式。修改者可以點選評論面板上的 "完成 "按鈕，以表示審查者的評論已被解決，或者點選 "Ack "按鈕，以確認評論已被閱讀，通常用於資訊性或選擇性評論。如果標註的評論未被解決，兩者都有解決的效果。這些快捷方式簡化了工作流程，減少了回覆評論所需的時間。

As mentioned earlier, comments are drafted as-you-go, but then “published” atomically, as shown in [Figure 19-7](#_bookmark1758). This allows authors and reviewers to ensure that they are happy with their comments before sending them out.

如前所述，評論是隨心所欲地起草的，但隨後以原子方式 "發表"，如圖19-7所示。這允許作者和審查者在傳送評論之前確保他們對自己的評論感到滿意。

![Figure 19-7](./images/Figure%2019-7.png)

*Figure 19-7. Preparing comments to the author*

### Understanding the State of a Change 瞭解變化的狀態

Critique provides a number of mechanisms to make it clear where in the comment- and-iterate phase a change is currently located. These include a feature for determining who needs to take action next, and a dashboard view of review/author status for all of the changes with which a particular developer is involved.

Critique提供了一些機制，使人們清楚地瞭解到某項修改目前處於評論和迭代階段的什麼位置。這些機制包括確定誰需要採取下一步行動的功能，以及特定開發者參與的所有修改的審查/作者狀態的儀表盤檢視。

#### “Whose turn” feature “輪到誰”功能

One important factor in accelerating the review process is understanding when it’s your turn to act, especially when there are multiple reviewers assigned to a change. This might be the case if the author wants to have their change reviewed by a software engineer and the user-experience person responsible for the feature, or the SRE carrying the pager for the service. Critique helps define who is expected to look at the change next by managing an *attention set* for each change.

加快審查過程的一個重要因素是瞭解什麼時候輪到你幹活了，特別是當有多個審查員被分配到一個變更時。如果作者想讓軟體工程師和負責該功能的使用者體驗人員審查他們的變更，或者為服務準備部署的SRE人員審查其更改，可能就是這種情況。透過管理每個變更的關注集，評論有助於確定下一個變更的關注者。

The attention set comprises the set of people on which a change is currently blocked. When a reviewer or author is in the attention set, they are expected to respond in a timely manner. Critique tries to be smart about updating the attention set when a user publishes their comments, but users can also manage the attention set themselves. Its usefulness increases even more when there are more reviewers in the change. The attention set is surfaced in Critique by rendering the relevant usernames in bold.

關注集由當前阻止更改的一組人組成。當評論者或作者在關注集中時，他們應該及時作出迴應。Critique自動化地在使用者發表評論時更新關注集，但使用者也可以自己管理關注集。當變化中的評論者較多時，它的作用就更大了。在Critique中，關注集是透過將相關的使用者名稱用黑體字顯示出來的。

After we implemented this feature, our users had a difficult time imagining the previous state. The prevailing opinion is: how did we get along without this? The alternative before we implemented this feature was chatting between reviewers and authors to understand who was dealing with a change. This feature also emphasizes the turn- based nature of code review; it is always at least one person’s turn to take action.

在我們實施這一功能後，我們的使用者很難想象以前的狀態。普遍的看法是：如果沒有這個，我們是怎麼過的？在我們實施這個功能之前，另一個選擇是審查員和作者之間的聊天，以瞭解誰在處理一個變化。這個功能也強調了程式碼審查的輪流性質；總是至少輪到一個人採取行動。

#### Dashboard and search system 儀表板和搜尋系統

Critique’s landing page is the user’s dashboard page, as depicted in [Figure 19-8](#_bookmark1762). The dashboard page is divided into user-customizable sections, each of them containing a list of change summaries.

Critique的登陸頁面是使用者的儀表盤頁面，如圖19-8所示。儀表盤頁面被分為使用者可訂製的部分，每個部分都包含一個變更摘要列表。

![Figure 19-8](./images/Figure%2019-8.png)

*Figure* *19-8. Dashboard view*

The dashboard page is powered by a search system called *Changelist Search*. Changelist Search indexes the latest state of all available changes (both pre- and post-submit) across all users at Google and allows its users to look up relevant changes by regular expression–based queries. Each dashboard section is defined by a query to Changelist Search. We have spent time ensuring Changelist Search is fast enough for interactive use; everything is indexed quickly so that authors and reviewers are not slowed down, despite the fact that we have an extremely large number of concurrent changes happening simultaneously at Google.

儀表板頁面是由一個名為*Changelist Search*的搜尋系統提供的。Changelist Search索引了 Google 所有使用者的所有可用變化的最新狀態（包括提交前和提交後），並允許其使用者透過基於正則表示式的查詢來查詢相關變化。每個儀表板部分都由對Changelist Search的查詢來定義。我們花了很多時間來確保Changelist Search搜尋足夠快；所有的東西都被快速索引，這樣作者和審稿人就不會被拖慢，儘管事實上 Google 同時出現了大量的併發更改。

To optimize the user experience (UX), Critique’s default dashboard setting is to have the first section display the changes that need a user’s attention, although this is customizable. There is also a search bar for making custom queries over all changes and browsing the results. As a reviewer, you mostly just need the attention set. As an author, you mostly just need to take a look at what is still waiting for review to see if you need to ping any changes. Although we have shied away from customizability in some other parts of the Critique UI, we found that users like to set up their dashboards differently without detracting from the fundamental experience, similar to the way everyone organizes their emails differently.[1](#_bookmark1770)

為了優化使用者體驗（UX），Critique的預設儀表盤設定是在第一部分顯示需要使用者關注的變更，不過這也是可以訂製的。還有一個搜尋欄，可以對所有修改進行自訂查詢，並瀏覽結果。作為一個審查員，你大多隻需要關注集。作為一個作者，你大多數時候只需要看一下哪些東西還在等待審查，看看你是否需要修正。儘管我們在Critique使用者介面的一些其他部分迴避了可訂製性，但我們發現使用者喜歡以不同的方式設定他們的儀表板，而不影響基本的體驗，就像每個人以不同的方式組織他們的電子郵件一樣。

## Stage 5: Change Approvals (Scoring a Change) 階段5：變更批准（對變更進行評分）

Showing whether a reviewer thinks a change is good boils down to providing concerns and suggestions via comments. There also needs to be some mechanism for providing a high-level “OK” on a change. At Google, the scoring for a change is divided into three parts:
•   LGTM (“looks good to me”)
•   Approval
•   The number of unresolved comments

顯示一個審查員是否認為一個變更是好的，歸根結底是透過評論提供關注和建議。此外，還需要有一些機制來提供一個高水平的 "OK"。在 Google ，對一個變化的打分分為三個部分：
- LGTM（“我覺得不錯”）
- 批准
- 未解決的評論的數量

An LGTM stamp from a reviewer means that “I have reviewed this change, believe that it meets our standards, and I think it is okay to commit it after addressing unresolved comments.” An Approval stamp from a reviewer means that “as a gatekeeper, I allow this change to be committed to the codebase.” A reviewer can mark comments as unresolved, meaning that the author will need to act upon them. When the change has at least one LGTM, sufficient approvals and no unresolved comments, the author can then commit the change. Note that every change requires an LGTM regardless of approval status, ensuring that at least two pairs of eyes viewed the change. This simple scoring rule allows Critique to inform the author when a change is ready to commit (shown prominently as a green page header).

審查者的LGTM印章意味著 "我已經審閱了這個變更，相信它符合我們的標準，我認為在解決了未解決的評論之後，可以提交它。” 審查者的批准標識意味著 "作為一個把關人，我允許這個修改被提交到程式碼函式庫中"。審查者可以將評論標記為未解決，這意味著作者需要對其採取行動。當變更至少有一個LGTM、足夠的批准和沒有未解決的評論時，作者可以提交變更。請注意，無論批准狀態如何，每項變更都需要一個LGTM，以確保至少有兩雙眼睛檢視該變更。這個簡單的評分規則使Critique可以在修改準備好提交時通知作者（以綠色頁首的形式突出顯示）。

```
1	Centralized “global” reviewers for large-scale changes (LSCs) are particularly prone to customizing this dashboard to avoid flooding it during an LSC (see Chapter 22).
1   大規模變更（LSC）的集中式“全域性”評審員特別傾向於訂製此儀表板，以避免在LSC期間將其淹沒（參見第22章）。
```

We made a conscious decision in the process of building Critique to simplify this rating scheme. Initially, Critique had a “Needs More Work” rating and also a “LGTM++”. The model we have moved to is to make LGTM/Approval always positive. If a change definitely needs a second review, primary reviewers can add comments but without LGTM/Approval. After a change transitions into a mostly-good state, reviewers will typically trust authors to take care of small edits—the tooling does not require repeated LGTMs regardless of change size.

在建立Critique的過程中，我們有意識地決定簡化這一評分方案。最初，Critique有一個 "需要更多工作 "的評級，也有一個 "LGTM++"。我們所採用的模式是使 `LGTM/批准` 總是積極的。如果變更確實需要第二次稽核，主要審查者可以新增內容，但無需LGTM/批准。在一個變化過渡到基本良好的狀態後，審查員通常會相信作者會處理好小的編輯--無論變更大小如何，該工具都不需要重複LGTM。

This rating scheme has also had a positive influence on code review culture. Reviewers cannot just thumbs-down a change with no useful feedback; all negative feedback from reviewers must be tied to something specific to be fixed (for example, an unresolved comment). The phrasing “unresolved comment” was also chosen to sound relatively nice.

這種評分方案也對程式碼審查文化產生了積極影響。審查者不能在沒有任何有用反饋的情況下對一個改動豎起大拇指；所有來自審查者的負面反饋都必須與需要修復的具體內容相聯絡（例如，一個未解決的評論）。選擇 "未解決的評論 "這一措辭也是為了聽起來比較好。

Critique includes a scoring panel, next to the analysis chips, with the following information:
•   Who has LGTM’ed the change
•   What approvals are still required and why
•   How many unresolved comments are still open

批評包括一個打分板，在分析卡片旁邊，有以下資訊。
- 誰進行了變更
- 還需要哪些批准，為什麼？
- 有多少的評論仍未解決

Presenting the scoring information this way helps the author quickly understand what they still need to do to get the change committed.

以這種方式呈現評分資訊有助於作者快速瞭解他們仍然需要做些什麼才能實現更改。

LGTM and Approval are *hard* requirements and can be granted only by reviewers. Reviewers can also revoke their LGTM and Approval at any time before the change is committed. Unresolved comments are *soft* requirements; the author can mark a comment “resolved” as they reply. This distinction promotes and relies on trust and communication between the author and the reviewers. For example, a reviewer can LGTM the change accompanied with unresolved comments without later on checking precisely whether the comments are truly addressed, highlighting the trust the reviewer places on the author. This trust is particularly important for saving time when there is a significant difference in time zones between the author and the reviewer. Exhibiting trust is also a good way to build trust and strengthen teams.

LGTM和批准是*硬性*要求，只能由審查者授予。在提交變更之前，審查者還可以隨時撤銷其LGTM和批准。未解決的評論是*軟*要求；作者可以在回覆時將評論標記為 "已解決"。這種區別促進並依賴於作者和審查者之間的信任和溝通。例如，審查者可以在LGTM的修改中伴隨著未解決的評論，而不需要後來精確地檢查這些評論是否真正被解決，這突出了審稿人對作者的信任。當作者和審稿人之間存在明顯的時區差異時，這種信任對於節省時間尤為重要。展現信任也是建立信任和加強團隊的一個好方法。

## Stage 6: Commiting a Change 階段6：提交變更

Last but not least, Critique has a button for committing the change after the review to avoid context-switching to a command-line interface.

最後但並非最不重要的是，Critique有一個在審查後提交修改的按鈕，以避免上下文切換到命令列介面。

### After Commit: Tracking History 提交後：追蹤歷史記錄

In addition to the core use of Critique as a tool for reviewing source code changes before they are committed to the repository, Critique is also used as a tool for change archaeology. For most files, developers can view a list of the past history of changes that modified a particular file in the Code Search system (see [Chapter 17](#_bookmark1485)), or navigate directly to a change. Anyone at Google can browse the history of a change to generally viewable files, including the comments on and evolution of the change. This enables future auditing and is used to understand more details about why changes were made or how bugs were introduced. Developers can also use this feature to learn how changes were engineered, and code review data in aggregate is used to produce trainings.

除了Critique的核心用途是在原始碼修改提交到版本函式庫之前對其進行審查外，Critique還被用作變更考古的工具。對於大多數檔案，開發者可以在程式碼搜尋系統中檢視過去修改某個檔案的歷史列表（見第17章），或者直接導航到某個修改。Google的任何人都可以瀏覽一般可檢視檔案的修改歷史，包括對修改的評論和演變。這使未來的審計成為可能，並被用來了解更多的細節，如為什麼會做出改變或如何引入bug。開發人員也可以使用這個功能來了解變化是如何被設計的，程式碼審查資料的彙總被用來製作培訓。

Critique also supports the ability to comment after a change is committed; for example, when a problem is discovered later or additional context might be useful for someone investigating the change at another time. Critique also supports the ability to roll back changes and see whether a particular change has already been rolled back.

Critique 還支援在修改提交後進行評論的能力；例如，當後來發現問題或額外的背景可能對另一個時間調查修改的人有用。Critique還支援回滾修改的能力，以及檢視某一修改是否已經被回滾。

------

Case Study: Gerrit 案例研究：Gerrit

Although Critique is the most commonly used review tool at Google, it is not the only one. Critique is not externally available due to its tight interdependencies with our large monolithic repository and other internal tools. Because of this, teams at Google that work on open source projects (including Chrome and Android) or internal projects that can’t or don’t want to be hosted in the monolithic repository use a different code review tool: Gerrit.

儘管Critique是Google最常用的審查工具，但它並不是唯一的工具。由於Critique與我們的大型單體函式庫和其他內部工具有緊密的相互依賴關係，所以Critique不能對外使用。正因為如此，在 Google 從事開源專案（包括Chrome和Android）或內部專案的團隊，如果不能或不想託管在單片函式庫中，就會使用另一種程式碼審查工具：Gerrit。

Gerrit is a standalone, open source code review tool that is tightly integrated with the Git version control system. As such, it offers a web UI to many Git features including code browsing, merging branches, cherry-picking commits, and, of course, code review. In addition, Gerrit has a fine-grained permission model that we can use to restrict access to repositories and branches.

Gerrit是一個獨立的開原始碼審查工具，與Git版本控制系統緊密整合。因此，它為許多Git特性提供了一個web UI，包括程式碼瀏覽、合併分支、提交，當然還有程式碼審查。此外，Gerrit有一個細粒度的許可權模型，我們可以使用它來限制對儲存函式庫和分支的訪問。

Both Critique and Gerrit have the same model for code reviews in that each commit is reviewed separately. Gerrit supports stacking commits and uploading them for individual review. It also allows the chain to be committed atomically after it’s reviewed.

Commission和Gerrit都有相同的程式碼評審模型，每個提交都是單獨評審的。Gerrit支援堆疊提交併將其上載以供個人審閱。它還允許在對鏈進行審查後以原子方式提交鏈

Being open source, Gerrit accommodates more variants and a wider range of use cases; Gerrit’s rich plug-in system enables a tight integration into custom environments. To support these use cases, Gerrit also supports a more sophisticated scoring system. A reviewer can veto a change by placing a –2 score, and the scoring system is highly configurable.

由於是開源的，Gerrit適應了更多的變體和更廣泛的用例；Gerrit豐富的外掛系統實現了與訂製環境的緊密整合。為了支援這些用例，Gerrit還支援更復雜的評分系統。評審員可以透過給-2分否決變更，評分系統是高度可配置的。

You can learn more about Gerrit and see it in action at [*https://www.gerritcodereview.com*](https://www.gerritcodereview.com/).

你可以在[*https://www.gerritcodereview.com*](https://www.gerritcodereview.com/)瞭解更多關於Gerrit的資訊，並看到它的執行情況。

------

## Conclusion 總結

There are a number of implicit trade-offs when using a code review tool. Critique builds in a number of features and integrates with other tools to make the review process more seamless for its users. Time spent in code reviews is time not spent coding, so any optimization of the review process can be a productivity gain for the company. Having only two people in most cases (author and reviewer) agree on the change before it can be committed keeps velocity high. Google greatly values the educational aspects of code review, even though they are more difficult to quantify.

在使用程式碼審查工具時，有一些隱含的權衡因素。Critique內建了許多功能，並與其他工具整合，使使用者的審查過程更加完美。花在程式碼評審上的時間並不是比花在編碼上的時間少多少，所以評審過程的任何優化都可以提高公司的生產效率。在大多數情況下，只有兩個人（作者和審查者）在提交修改前達成一致，可以保持高速度。 Google 非常重視程式碼審查的培訓方面，儘管它們更難以量化。

To minimize the time it takes for a change to be reviewed, the code review process should flow seamlessly, informing users succinctly of the changes that need their attention and identifying potential issues before human reviewers come in (issues are caught by analyzers and Continuous Integration). When possible, quick analysis results are presented before the longer-running analyses can finish.

為了最大限度地減少評審更改所需的時間，程式碼評審過程應該無縫流動，簡潔地告知使用者需要關注的更改，並在人工評審員介入之前確定潛在問題（問題由分析人員和持續整合人員發現）。如果可能，在較長時間執行的分析完成之前，會顯示快速分析結果。

There are several ways in which Critique needs to support questions of scale. The Critique tool must scale to the large quantity of review requests produced without suffering a degradation in performance. Because Critique is on the critical path to getting changes committed, it must load efficiently and be usable for special situations such as unusually large changes.[2](#_bookmark1778) The interface must support managing user activities (such as finding relevant changes) over the large codebase and help reviewers and authors navigate the codebase. For example, Critique helps with finding appropriate reviewers for a change without having to figure out the ownership/maintainer landscape (a feature that is particularly important for large-scale changes such as API migrations that can affect many files).

Critique需要在幾個方面支援規模問題。Critique工具必須在不降低效能的情況下，適應大量的審查請求。由於Critique是在提交修改的關鍵路徑上，它必須有效地載入，並能在特殊情況下使用，如異常大的修改。介面必須支援在大型程式碼函式庫中管理使用者活動（如尋找相關修改），並幫助評審員和作者瀏覽程式碼函式庫。例如，Critique有助於為某一變更找到合適的審查者，而不必弄清所有權/維護者的情況（這一功能對於大規模的變更，如可能影響許多檔案的API遷移，尤為重要）。

Critique favors an opinionated process and a simple interface to improve the general review workflow. However, Critique does allow some customizability: custom analyzers and presubmits provide specific context on changes, and some team-specific policies (such as requiring LGTM from multiple reviewers) can be enforced.

Critique傾向於採用意見一致的流程和簡單的介面來改善一般的審查工作流程。然而，Critique確實允許一些自訂功能：自訂分析器和預提交提供了具體的修改內容，而且可以強制執行一些特定的團隊策略（如要求多個審稿人提供LGTM）。

```
2	Although most changes are small (fewer than 100 lines), Critique is sometimes used to review large refactoring changes that can touch hundreds or thousands of files, especially for LSCs that must be executed atomically (see Chapter 22).
2 儘管大多數改動都很小（少於100行），但Critique有時也被用來審查大型的重構改動，這些改動可能會觸及成百上千個檔案，特別是對於那些必須原子化執行的LSCs（見第22章）。
```

Trust and communication are core to the code review process. A tool can enhance the experience, but can’t replace them. Tight integration with other tools has also been a key factor in Critique’s success.

信任和溝通是程式碼審查過程的核心。工具可以增強體驗，但不能替代它們。與其他工具的緊密結合也是Critique成功的一個關鍵因素。

## TL;DRs  內容提要

•   Trust and communication are core to the code review process. A tool can enhance the experience, but it can’t replace them.
•   Tight integration with other tools is key to great code review experience.
•   Small workflow optimizations, like the addition of an explicit “attention set,” can increase clarity and reduce friction substantially.

- 信任和溝通是程式碼審查過程的核心。工具可以增強體驗，但不能替代它們。
- 與其他工具的緊密整合是獲得優秀程式碼審查體驗的關鍵。
- 小的工作流程優化，如增加一個明確的 "關注集"，可以提高清晰度並大大減少摩擦。

