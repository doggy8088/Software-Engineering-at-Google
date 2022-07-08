
**CHAPTER 23**

# Continuous Integration

# 第二十三章 持續整合

                                    Written by Rachel Tannenbaum Edited by Lisa Carey

*Continuous Integration*, or CI, is generally defined as “a software development practice where members of a team integrate their work frequently [...] Each integration is verified by an automated build (including test) to detect integration errors as quickly as possible.”[^1] Simply put, the fundamental goal of CI is to automatically catch problematic changes as early as possible.

*持續整合*，或CI，通常被定義為 "一種軟體開發實踐，團隊成員經常整合他們的工作[......]每個整合都由自動建構（包括測試）來驗證，以儘快發現整合錯誤。"1簡單地說，CI的基本目標是儘可能早地自動捕捉有問題的變化。

In practice, what does “integrating work frequently” mean for the modern, distributed application? Today’s systems have many moving pieces beyond just the latest versioned code in the repository. In fact, with the recent trend toward microservices, the changes that break an application are less likely to live inside the project’s immediate codebase and more likely to be in loosely coupled microservices on the other side of a network call. Whereas a traditional continuous build tests changes in your binary, an extension of this might test changes to upstream microservices. The dependency is just shifted from your function call stack to an HTTP request or Remote Procedure Calls (RPC).

在實踐中，"頻繁地整合工作 "對於現代的、分散式的應用程式意味著什麼？今天的系統除了儲存函式庫中最新版本的程式碼外，還有許多可移動的部分。事實上，隨著近來的微服務趨勢，破壞應用程式的變化不太可能存在於專案的即時程式碼函式庫中，而更可能存在於網路呼叫的另一端的鬆散耦合的微服務中。傳統的持續建構是測試二進位制檔案的變更，而其延伸則是測試上游微服務的變化。依賴性只是從你的函式呼叫棧轉移到HTTP請求或遠端過程呼叫（RPC）。

Even further from code dependencies, an application might periodically ingest data or update machine learning models. It might execute on evolving operating systems, runtimes, cloud hosting services, and devices. It might be a feature that sits on top of a growing platform or be the platform that must accommodate a growing feature base. All of these things should be considered dependencies, and we should aim to “continuously integrate” their changes, too. Further complicating things, these changing components are often owned by developers outside our team, organization, or company and deployed on their own schedules.

甚至在程式碼依賴性之外，應用程式可能會定期接收資料或更新機器學習模型。它可能在不斷髮展的作業系統、執行時、雲託管服務和裝置上執行。它可能是位於不斷增長的平臺之上的功能，也可能是必須適應不斷增長的功能基礎的平臺。所有這些都應該被視為依賴關係，我們也應該致力於“持續整合”它們的變化。更復雜的是，這些變化的元件通常由我們團隊、組織或公司之外的開發人員擁有，並按照他們自己的時間表部署。

> [^1]:	https://www.martinfowler.com/articles/continuousIntegration.html


So, perhaps a better definition for CI in today’s world, particularly when developing at scale, is the following:

*Continuous Integration (2)*: the continuous assembling and testing of our entire complex and rapidly evolving ecosystem.

因此，在當今世界，特別是在大規模開發時，對CI更好的定義也許是以下幾點：

·持續整合：對我們整個複雜和快速發展的生態系統進行持續的組裝和測試。

It is natural to conceptualize CI in terms of testing because the two are tightly coupled, and we’ll do so throughout this chapter. In previous chapters, we’ve discussed a comprehensive range of testing, from unit to integration, to larger-scoped systems.

從測試的角度對CI進行思考是很自然的，因為兩者緊密結合，我們將在本章中這樣做。在前面的章節中，我們討論了一系列全面的測試，從單元到整合，再到更大範圍的系統。

From a testing perspective, CI is a paradigm to inform the following:
- *Which* tests to run *when* in the development/release workflow, as code (and other) changes are continuously integrated into it
- *How* to compose the system under test (SUT) at each point, balancing concerns like fidelity and setup cost

從測試的角度來看，CI是一種正規化，可以告知以下內容：
- 在開發/發佈工作流程中，由於程式碼（和其他）變化不斷地被整合到其中，在什麼時候執行哪些測試
- 如何在每個點上組成被測系統（SUT），平衡模擬度和設定成本等問題

For example, which tests do we run on presubmit, which do we save for post-submit, and which do we save even later until our staging deploy? Accordingly, how do we represent our SUT at each of these points? As you might imagine, requirements for a presubmit SUT can differ significantly from those of a staging environment under test. For example, it can be dangerous for an application built from code pending review on presubmit to talk to real production backends (think security and quota vulnerabilities), whereas this is often acceptable for a staging environment.

例如，我們在預提交上執行哪些測試，在提交後儲存哪些測試，哪些甚至要儲存到我們的臨時部署？因此，我們如何在這些點上表示SUT？正如你所想象的，預提交SUT的需求可能與測試中的部署環境的需求有很大的不同。例如，從預提交的待審程式碼建構的應用程式與真正的生產後端對話可能是危險的（考慮安全和配額漏洞），而這對於臨時環境來說通常是可以接受的。

And *why* should we try to optimize this often-delicate balance of testing “the right things” at “the right times” with CI? Plenty of prior work has already established the benefits of CI to the engineering organization and the overall business alike.[^2] These outcomes are driven by a powerful guarantee: verifiable—and timely—proof that the application is good to progress to the next stage. We don’t need to just hope that all contributors are very careful, responsible, and thorough; we can instead guarantee the working state of our application at various points from build throughout release, thereby improving confidence and quality in our products and productivity of our teams.

為什麼我們要嘗試用 CI 來優化在 "正確的時間 "測試 "正確的事情 "的這種往往很微妙的平衡？這些結果是由一個強有力的保證所驅動的：可驗證的、及時的、可證明應用程式可以進入下一階段的證明。我們不需要僅僅希望所有的貢獻者都非常謹慎、負責和閉環；相反，我們可以保證我們的應用程式在從建構到發佈的各個階段的工作狀態，從而提高對我們產品的信心和品質以及我們團隊的生產力。

In the rest of this chapter, we’ll introduce some key CI concepts, best practices and challenges, before looking at how we manage CI at Google with an introduction to our continuous build tool, TAP, and an in-depth study of one application’s CI transformation.

在本章的其餘部分中，我們將介紹一些關鍵CI概念、最佳實踐和挑戰，然後介紹我們如何在Google管理CI，並介紹我們的持續建構工具TAP，以及對某個應用程式的CI轉換的深入研究。


> [^2]:	Forsgren, Nicole, et al. (2018). Accelerate: The Science of Lean Software and DevOps: Building and Scaling High Performing Technology Organizations. IT Revolution./
> 2   Forsgren，Nicole等人（2018年）。加速：精益軟體科學和DevOps：建立和擴充套件高效能技術組織。這是一場革命。



## CI Concepts CI概念

First, let’s begin by looking at some core concepts of CI.

首先，讓我們先看看CI的一些核心概念。

### Fast Feedback Loops 快速反饋迴路

As discussed in [Chapter 11](#_bookmark838), the cost of a bug grows almost exponentially the later it is caught. [Figure 23-1 ](#_bookmark2031)shows all the places a problematic code change might be caught in its lifetime.

正如第11章所討論的，一個bug被捕獲的時間越晚，其成本幾乎呈指數增長。圖23-1顯示了有問題的程式碼更改在其生命週期中可能出現的所有位置。

![Figure 23-1](./images/Figure%2023-1.png)

*Figure* *23-1.* *Life* *of* *a* *code* *change*

In general, as issues progress to the “right” in our diagram, they become costlier for the following reasons:

•   They must be triaged by an engineer who is likely unfamiliar with the problematic code change.

•   They require more work for the code change author to recollect and investigate the change.

•   They negatively affect others, whether engineers in their work or ultimately the end user.

一般來說，隨著問題向我們圖中的 "右側 "發展，它們的成本會變得更高，原因如下：

- 它們必須由可能不熟悉問題程式碼更改的工程師來處理。

- 這些問題需要程式碼修改者做更多的工作來回憶和調查這些修改。

- 它們會對其他人產生負面影響，無論是工作中的工程師還是最終的終端使用者。

To minimize the cost of bugs, CI encourages us to use *fast feedback loops.*[^3] Each time we integrate a code (or other) change into a testing scenario and observe the results, we get a new *feedback loop*. Feedback can take many forms; following are some common ones (in order of fastest to slowest):
- The edit-compile-debug loop of local development
- Automated test results to a code change author on presubmit
- An integration error between changes to two projects, detected after both are submitted and tested together (i.e., on post-submit)
- An incompatibility between our project and an upstream microservice dependency, detected by a QA tester in our staging environment, when the upstream service deploys its latest changes
- Bug reports by internal users who are opted in to a feature before external users
- Bug or outage reports by external users or the press

為了使bug的代價最小化，CI鼓勵我們使用*快速反饋環*。每次我們將程式碼（或其他）變化整合到測試場景中並觀察結果時，我們就會得到一個新的*反饋迴路*。反饋可以有很多形式；下面是一些常見的形式（按從快到慢的順序）:
- 本地開發的編輯-編譯-除錯迴路
- 在提交前向程式碼修改者提供自動測試結果
- 兩個專案變更之間的整合錯誤，在兩個專案一起提交和測試後檢測（即提交後）。
- 當上遊服務部署其最新變化時，我們的專案和上游微服務的依賴關係之間的不相容，由我們臨時環境中的QA測試員發現。
- 在外部使用者之前使用功能的內部使用者的錯誤報告
- 外部使用者或媒體的錯誤或故障報告

> [^3]:	This is also sometimes called “shifting left on testing.”/
> 3 這有時也被稱為 "測試左移"。


*Canarying*—or deploying to a small percentage of production first—can help minimize issues that do make it to production, with a subset-of-production initial feedback loop preceding all-of-production. However, canarying can cause problems, too, particularly around compatibility between deployments when multiple versions are deployed at once. This is sometimes known as *version skew*, a state of a distributed system in which it contains multiple incompatible versions of code, data, and/or configuration. Like many issues we look at in this book, version skew is another example of a challenging problem that can arise when trying to develop and manage software over time.

*金絲雀*-或者先部署到一小部分生產，可以有助於最小化減少進入生產的問題，在所有生產之前先部署一部分生產初始反饋迴路。但是，金絲雀部署也會導致新的問題，尤其是在同時部署多個版本時，部署之間的相容性問題。這有時被稱為版本傾斜，分散式系統的一種狀態，其中包含多個不相容的程式碼、資料和/或配置版本。就像我們在本書中看到的許多問題一樣，版本傾斜是另一個在嘗試開發和管理軟體時可能出現的具有挑戰性的問題的例子。

*Experiments* and *feature flags* are extremely powerful feedback loops. They reduce deployment risk by isolating changes within modular components that can be dynamically toggled in production. Relying heavily on feature-flag-guarding is a common paradigm for Continuous Delivery, which we explore further in [Chapter 24](#_bookmark2100).

*實驗*特性標誌是非常強大的反饋迴路。它們透過隔離模組化元件中可以在生產中動態切換的更改來降低部署風險。嚴重依賴功能標誌保護是持續交付的常見範例，我們將在第24章中進一步探討。

#### Accessible and actionable feedback 可獲取和可操作的反饋

It’s also important that feedback from CI be widely accessible. In addition to our open culture around code visibility, we feel similarly about our test reporting. We have a unified test reporting system in which anyone can easily look up a build or test run, including all logs (excluding user Personally Identifiable Information [PII]), whether for an individual engineer’s local run or on an automated development or staging build.

同樣重要的是，來自CI的反饋可以被廣泛獲取。除了我們圍繞程式碼可見性的開放文化之外，我們對我們的測試報告也有類似的方式。我們有一個統一的測試報告系統，任何人都可以很容易地檢視建構或測試執行，包括所有的日誌（不包括使用者的個人身份資訊[PII]），無論是個人工程師的本地執行還是自動化開發或分段建構。

Along with logs, our test reporting system provides a detailed history of when build or test targets began to fail, including audits of where the build was cut at each run, where it was run, and by whom. We also have a system for flake classification, which uses statistics to classify flakes at a Google-wide level, so engineers don’t need to figure this out for themselves to determine whether their change broke another project’s test (if the test is flaky: probably not).

除了日誌，我們的測試報告系統還提供了建構或測試目標開始失敗的詳細歷史記錄，包括每次執行時在何處剪下建構、在何處執行以及由誰執行的審計日誌。我們還有一個薄片分類系統，該系統使用統計資料在谷歌範圍內對薄片進行分類，因此工程師不需要自己來確定他們的更改是否破壞了另一個專案的測試（如果測試是薄片：可能不是）。

Visibility into test history empowers engineers to share and collaborate on feedback, an essential requirement for disparate teams to diagnose and learn from integration failures between their systems. Similarly, bugs (e.g., tickets or issues) at Google are open with full comment history for all to see and learn from (with the exception, again, of customer PII).

對測試歷史的可視性使工程師能夠就反饋進行共享和協作，這是不同團隊診斷和學習系統間整合故障的基本要求。類似地，谷歌的bug（如罰單或問題）是開放的，有完整的評論歷史供所有人檢視和學習（客戶PII除外）。

Finally, any feedback from CI tests should not just be accessible but actionable—easy to use to find and fix problems. We’ll look at an example of improving user-unfriendly feedback in our case study later in this chapter. By improving test output readability, you automate the understanding of feedback.

最後，來自CI測試的任何反饋不僅應該是可訪問的，而且應該是可操作的--易於用來發現和修復問題。我們將在本章後面的案例研究中看一個改進使用者不友好反饋的例子。透過改善測試輸出的可讀性，你可以自動理解反饋。

### Automation 自動化

It’s well known that [automating development-related tasks saves engineering resources](https://oreil.ly/UafCh)in the long run. Intuitively, because we automate processes by defining them as code, peer review when changes are checked in will reduce the probability of error. Of course, automated processes, like any other software, will have bugs; but when implemented effectively, they are still faster, easier, and more reliable than if they were attempted manually by engineers.

眾所周知，從長遠來看，開發相關任務的自動化可以節省工程資源。直觀地說，因為我們透過將流程定義為程式碼來實現自動化，所以在修改時的同行評審將減少錯誤的概率。當然，自動化流程，像其他軟體一樣，會有錯誤；但如果有效地實施，它們仍然比工程師手動嘗試更快，更容易，更可靠。

CI, specifically, automates the *build* and *release* processes, with a Continuous Build and Continuous Delivery. Continuous testing is applied throughout, which we’ll look at in the next section.

特別是CI，它使*建構*和*發佈*過程自動化，有持續建構和持續交付。持續測試貫穿始終，我們將在下一節中介紹。

#### Continuous Build 連續建構

The *Continuous Build* (CB) integrates the latest code changes at head[^4] and runs an automated build and test. Because the CB runs tests as well as building code, “breaking the build” or “failing the build” includes breaking tests as well as breaking compilation.

持續建構（CB）集成了最新的程式碼修改，並執行自動建構和測試。因為CB在執行測試的同時也在建構程式碼，"破壞建構 "或 "建構失敗 "包括破壞測試和破壞編譯。

After a change is submitted, the CB should run all relevant tests. If a change passes all tests, the CB marks it passing or “green,” as it is often displayed in user interfaces (UIs). This process effectively introduces two different versions of head in the repository: *true head*, or the latest change that was committed, and *green head,* or the latest change the CB has verified. Engineers are able to sync to either version in their local development. It’s common to sync against green head to work with a stable environment, verified by the CB, while coding a change but have a process that requires changes to be synced to true head before submission.

提交變更後，CB應執行所有相關測試。如果更改通過了所有測試，CB會將其標記為透過或綠色”，因為它通常顯示在使用者介面（UI）中。該流程有效地在報告中引入了兩種不同版本的head：真實head或已提交的最新變更，以及綠色head或CB已驗證的最新變更。工程師可以在本地開發中同步到任一版本。在編寫變更程式碼時，通常會與綠色head同步，以便在穩定的環境中工作，並經CB驗證，但有一個流程要求在提交變更之前將變更同步到真實head。

#### Continuous Delivery 連續交付

The first step in Continuous Delivery (CD; discussed more fully in [Chapter 24](#_bookmark2100)) is *release automation*, which continuously assembles the latest code and configuration from head into release candidates. At Google, most teams cut these at green, as opposed to true, head.

持續交付（CD；在第24章中詳細討論）的第一步是發佈自動化，它不斷地將最新的程式碼和配置從head組裝成候選發佈版本。在谷歌，大多數團隊都是在綠色（而不是真正的）head進行切割。

*Release candidate* (RC): A cohesive, deployable unit created by an automated process,[^5] assembled of code, configuration, and other dependencies that have passed the continuous build.

*候選版本*（RC）:由自動化流程建立的內聚、可部署單元，由透過持續建構的程式碼、配置和其他依賴關係組成。



Note that we include configuration in release candidates—this is extremely important, even though it can slightly vary between environments as the candidate is promoted. We’re not necessarily advocating you compile configuration into your binaries—actually, we would recommend dynamic configuration, such as experiments or feature flags, for many scenarios.[^6]

請注意，我們在候選版本中包含了配置--這一點極為重要，儘管在候選版本的推廣過程中，不同環境下的配置會略有不同。我們不一定提倡你把配置編譯到你的二進位制檔案中--事實上，我們建議在許多情況下使用動態配置，如實驗或特徵標誌。

Rather, we are saying that any static configuration you *do* have should be promoted as part of the release candidate so that it can undergo testing along with its corresponding code. Remember, a large percentage of production bugs are caused by “silly” configuration problems, so it’s just as important to test your configuration as it is your code (and to test it along *with* the same code that will use it). Version skew is often caught in this release-candidate-promotion process. This assumes, of course, that your static configuration is in version control—at Google, static configuration is in version control along with the code, and hence goes through the same code review process.

相反，我們的意思是，您所擁有的任何靜態配置都應該作為候選版本的一部分進行升級，以便它可以與其對應的程式碼一起接受測試。記住，很大比例的生產錯誤是由 "愚蠢的 "配置問題引起的，所以測試你的配置和測試你的程式碼一樣重要（而且要和將要使用它的相同程式碼一起測試）。在這個發佈--候選--推廣的過程中，經常會出現版本傾斜。當然，這是假設你的靜態配置是在版本控制中的--在谷歌，靜態配置是和程式碼一起在版本控制中的，因此要經過同樣的程式碼審查過程。

We then define CD as follows:
	*Continuous Delivery* (CD): a continuous assembling of release candidates, followed by the promotion and testing of those candidates throughout a series of environments— sometimes reaching production and sometimes not.

那麼我們對CD的定義如下:
	*持續交付（CD）*：持續集合候選版本，然後在一系列環境中推廣和測試這些候選版本--有時達到生產階段，有時不達到生產階段。

The promotion and deployment process often depends on the team. We’ll show how our case study navigated this process.

升級和部署過程通常取決於團隊。我們將展示我們的案例研究如何引導這一過程。

For teams at Google that want continuous feedback from new changes in production (e.g., Continuous Deployment), it’s usually infeasible to continuously push entire binaries, which are often quite large, on green. For that reason, doing a *selective* Continuous Deployment, through experiments or feature flags, is a common strategy.[^7]

對於谷歌的團隊來說，他們希望從生產中的新變化（例如，持續部署）中獲得持續的反饋，通常不可能持續地將整個二進位制檔案（通常相當大）推到綠色上。因此，透過實驗或特性標誌進行選擇性連續部署是一種常見的策略。

As an RC progresses through environments, its artifacts (e.g., binaries, containers) ideally should not be recompiled or rebuilt. Using containers such as Docker helps enforce consistency of an RC between environments, from local development onward. Similarly, using orchestration tools like Kubernetes (or in our case, usually [Borg](https://oreil.ly/89yPv)), helps enforce consistency between deployments. By enforcing consistency of our release and deployment between environments, we achieve higher-fidelity earlier testing and fewer surprises in production.

當一個RC在各種環境中發展，它的建構（如二進位制檔案、容器）最好不要被重新編譯或重建。使用像Docker這樣的容器有助於在不同的環境中強制執行RC的一致性，從本地開發開始。同樣，使用像Kubernetes這樣的協調工具（或者在我們的例子中，通常是[Borg](https://oreil.ly/89yPv)），有助於強制執行部署之間的一致性。透過強制執行在不同環境間的發佈和部署的一致性，我們實現了更高的保真度、更早的測試和更少的生產意外。

> 4	Head is the latest versioned code in our monorepo. In other workflows, this is also referred to as master, mainline, or trunk. Correspondingly, integrating at head is also known as trunk-based development./
> 4 Head是我們monorepo中最新版本的程式碼。在其他工作流程中，這也被稱為主幹、主線或主幹。相應地，在head整合也被稱為基於主幹的開發。
>
> 5	At Google, release automation is managed by a separate system from TAP. We won’t focus on how release automation assembles RCs, but if you’re interested, we do refer you to Site Reliability Engineering (O’Reilly) in which our release automation technology (a system called Rapid) is discussed in detail./
> 5 在谷歌，發佈自動化是由一個獨立於TAP的系統管理的。我們不會專注於發佈自動化是如何組裝RC的，但如果你有興趣，我們會向你推薦《網站可靠性工程》（O'Reilly），其中詳細討論了我們的發佈自動化技術（一個叫做Rapid的系統）。
>
> 6	CD with experiments and feature flags is discussed further in Chapter 24./
> 6 第24章進一步討論了帶有實驗和特徵標誌的CD。
>
> 7	We call these “mid-air collisions” because the probability of it occurring is extremely low; however, when this does happen, the results can be quite surprising./
> 7 我們稱這些為 "空中碰撞"，因為它發生的概率極低；然而，當這種情況發生時，其結果可能是相當令人驚訝的。


### Continuous Testing 持續測試

Let’s look at how CB and CD fit in as we apply Continuous Testing (CT) to a code change throughout its lifetime, as shown [Figure 23-2](#_bookmark2049).

讓我們來看看，當我們將持續測試（CT）應用於程式碼變更的整個生命週期時，CB和CD是如何配合的，如圖23-2所示。

![Figure 23-2](./images/Figure%2023-2.png)

*Figure* *23-2.* *Life* *of* *a* *code* *change* *with* *CB* *and* *CD*

The rightward arrow shows the progression of a single code change from local development to production. Again, one of our key objectives in CI is determining *what* to test *when* in this progression. Later in this chapter, we’ll introduce the different testing phases and provide some considerations for what to test in presubmit versus post-submit, and in the RC and beyond. We’ll show that, as we shift to the right, the code change is subjected to progressively larger-scoped automated tests.

向右箭頭顯示單個程式碼更改從本地開發到生產的過程。同樣，我我們在CI中的一個關鍵目標是確定在這個過程中*什麼時候*測試什麼。在本章後面，我們將介紹不同的測試階段，並就提交前與提交後以及RC和其他階段測試內容的注意事項。我們將展示，當我們向右移動時，程式碼更改將受到範圍越來越大的自動化測試的影響。

#### Why presubmit isn’t enough 為什麼僅靠預提交還不夠的

With the objective to catch problematic changes as soon as possible and the ability to run automated tests on presubmit, you might be wondering: why not just run all tests on presubmit?

為了儘快發現有問題的更改，並且能夠在預提交上執行自動測試，你可能會想：為什麼不在預提交時執行所有測試？

The main reason is that it’s too expensive. Engineer productivity is extremely valuable, and waiting a long time to run every test during code submission can be severely disruptive. Further, by removing the constraint for presubmits to be exhaustive, a lot of efficiency gains can be made if tests pass far more frequently than they fail. For example, the tests that are run can be restricted to certain scopes, or selected based on a model that predicts their likelihood of detecting a failure.

主要原因是它成本太高了。工程師的工作效率是非常寶貴的，在提交程式碼期間等待很長時間執行每個測試可能會嚴重降低生產力。此外，透過取消對預提交的限制，如果測試透過的頻率遠遠高於失敗的頻率，就可以獲得大量的效率提升。例如，執行的測試可以限制在特定範圍內，或者根據預測其檢測故障可能性的模型進行選擇。

Similarly, it’s expensive for engineers to be blocked on presubmit by failures arising from instability or flakiness that has nothing to do with their code change.

同樣，如果工程師在提交前被與他們的程式碼修改無關的不穩定或軟弱性引起的故障所阻擋，代價也很高。

Another reason is that during the time we run presubmit tests to confirm that a change is safe, the underlying repository might have changed in a manner that is incompatible with the changes being tested. That is, it is possible for two changes that touch completely different files to cause a test to fail. We call this a mid-air collision,and though generally rare, it happens most days at our scale. CI systems for smaller repositories or projects can avoid this problem by serializing submits so that there is no difference between what is about to enter and what just did.

另一個原因是，在我們執行預提交測試以確認更改是安全的過程中，底層儲存函式庫可能以與正在測試的更改不相容的方式進行了更改。也就是說，兩個涉及完全不同檔案的更改可能會導致測試失敗。我們稱這種情況為空中碰撞，雖然一般來說很少發生，但大多數情況下都會發生在我們的掌控範圍內。用於較小儲存函式庫或專案的CI系統可以透過序列化提交來避免此問題，以便在即將輸入的內容和剛剛輸入的內容之間沒有區別。

#### Presubmit versus post-submit 預提交與提交後

So, which tests *should* be run on presubmit? Our general rule of thumb is: only fast, reliable ones. You can accept some loss of coverage on presubmit, but that means you need to catch any issues that slip by on post-submit, and accept some number of rollbacks. On post-submit, you can accept longer times and some instability, as long as you have proper mechanisms to deal with it.

那麼，哪些測試*應該*在預提交時執行？我們的一般經驗法則是：只有快速、可靠的測試。你可以接受在預提交時有一些覆蓋面的損失，但這意味著你需要在提交後抓住任何漏掉的問題，並接受一定的回滾的次數。在提交後，你可以接受更長的時間和一些不穩定性，只要你有適當的機制來處理它。

We don’t want to waste valuable engineer productivity by waiting too long for slow tests or for too many tests—we typically limit presubmit tests to just those for the project where the change is happening. We also run tests concurrently, so there is a resource decision to consider as well. Finally, we don’t want to run unreliable tests on presubmit, because the cost of having many engineers affected by them, debugging the same problem that is not related to their code change, is too high.

我們不想因為等待太長時間的緩慢測試或太多測試而浪費寶貴的工程師生產力--我們通常將預提交的測試限制在發生變化的專案上。我們還同時執行測試，所以也要考慮資源決定。最後，我們不希望在預提交時執行不可靠的測試，因為讓許多工程師受其影響，除錯與他們的程式碼變更無關的同一個問題的成本太高。

Most teams at Google run their small tests (like unit tests) on presubmit[^8]—these are the obvious ones to run as they tend to be the fastest and most reliable. Whether and how to run larger-scoped tests on presubmit is the more interesting question, and this varies by team. For teams that do want to run them, hermetic testing is a proven approach to reducing their inherent instability. Another option is to allow large- scoped tests to be unreliable on presubmit but disable them aggressively when they start failing.

谷歌的大多數團隊都在預提交上執行他們的小型測試（如單元測試）--這些是明顯要執行的，因為它們往往是最快和最可靠的。是否以及如何在提交前執行更大範圍的測試是個更有趣的問題，這因團隊而異。對於想要執行這些測試的團隊來說，封閉測試是一種行之有效的方法來減少其固有的不穩定性。另一個選擇是允許大範圍的測試在預提交時不可靠，但當它們開始失敗時，要主動禁用它們。

> [^8]:	Each team at Google configures a subset of its project’s tests to run on presubmit (versus post-submit). In reality, our continuous build actually optimizes some presubmit tests to be saved for post-submit, behind the scenes. We’ll further discuss this later on in this chapter./
> 8 谷歌的每個團隊都將其專案的測試的一個子集配置為在預提交執行（相對於提交後）。實際上，我們的持續建構實際上在幕後優化了一些預提交的測試，以儲存到提交後。我們將在本章的後面進一步討論這個問題。

#### Release candidate testing 候選版本測試

After a code change has passed the CB (this might take multiple cycles if there were failures), it will soon encounter CD and be included in a pending release candidate.

在程式碼修改透過CB（如果有失敗的話，這可能需要多個週期）後，它很快再進行CD，並被納入待發布的候選版本。

As CD builds RCs, it will run larger tests against the entire candidate. We test a release candidate by promoting it through a series of test environments and testing it at each deployment. This can include a combination of sandboxed, temporary environments and shared test environments, like dev or staging. It’s common to include some manual QA testing of the RC in shared environments, too.

在CD建構RC的過程中，它將針對整個候選版本執行更大範圍測試。我們透過一系列的測試環境來測試候選發佈版，並在每次部署時對其進行測試。這可能包括沙盒、臨時環境和共享測試環境的組合，如開發或臨時。通常也包括在共享環境中對RC的一些手動QA測試。

There are several reasons why it’s important to run a comprehensive, automated test suite against an RC, even if it is the same suite that CB just ran against the code on post-submit (assuming the CD cuts at green):

- *As a sanity check*  
​	We double check that nothing strange happened when the code was cut and recompiled in the RC.

- *For* *auditability*  
​	If an engineer wants to check an RC’s test results, they are readily available and associated with the RC, so they don’t need to dig through CB logs to find them.

- *To allow for cherry picks*  
​	If you apply a cherry-pick fix to an RC, your source code has now diverged from the latest cut tested by the CB.

- *For emergency pushes*  
​	In that case, CD can cut from true head and run the minimal set of tests necessary to feel confident about an emergency push, without waiting for the full CB to pass.

有幾個原因可以說明為什麼對RC執行一個全面的、自動化的測試套件很重要，即使它是CB在提交後對程式碼執行的同一個套件（假設CD是綠色的）:

- *作為理性的檢查*  
​	我們仔細檢查，當代碼在RC中被切割和重新編譯時，確保沒有任何奇怪的事情發生。

- *為了便於審計*  
​	如果工程師想檢查RC的測試結果，他們很容易得到，並與RC相關聯，所以他們不需要在CB日誌中尋找它們。

- *允許偷樑換柱*  
​	如果你對一個RC應用了偷樑換柱式的修復，你的原始碼現在已經與CB測試的最新版本相去甚遠。

- *用於緊急推送*  
​	在這種情況下，CD可以從真正的head切分，並執行必要的最小的測試集，對緊急推送感到有信心，而不等待完整的CB透過。

#### Production testing 生產測試

Our continuous, automated testing process goes all the way to the final deployed environment: production. We should run the same suite of tests against production (sometimes called *probers*) that we did against the release candidate earlier on to verify: 1) the working state of production, according to our tests, and 2) the relevance of our tests, according to production.

我們的持續、自動化測試過程一直持續到最後的部署環境：生產環境。我們應該對生產環境執行相同的測試套件（有時稱為*probers*），就像我們早期對候選發佈版所做的那樣，以驗證：1）根據我們的測試，生產環境的工作狀態；2）根據生產環境，我們測試的相關性。

Continuous testing at each step of the application’s progression, each with its own trade-offs, serves as a reminder of the value in a “defense in depth” approach to catching bugs—it isn’t just one bit of technology or policy that we rely upon for quality and stability, it’s many testing approaches combined.

在應用程式進展的每一步進行持續測試，每一步都有其自身的權衡，這提醒了 "深度防禦 "方法在捕捉錯誤方面的價值--我們依靠的不僅僅是一種技術或策略來保證品質和穩定性，還有多種測試方法的結合。

-----

CI Is Alerting CI正在告警
Titus Winters

As with responsibly running production systems, sustainably maintaining software systems also requires continual automated monitoring. Just as we use a monitoring and alerting system to understand how production systems respond to change, CI reveals how our software is responding to changes in its environment. Whereas production monitoring relies on passive alerts and active probers of running systems, CI uses unit and integration tests to detect changes to the software before it is deployed. Drawing comparisons between these two domains lets us apply knowledge from one to the other.

與負責任地執行生產系統一樣，可持續地維護軟體系統也需要持續的自動監控。正如我們使用監控和告警系統來了解生產系統對變化的反應一樣，CI揭示了我們的軟體是如何對其環境的變化做出反應的。生產監控依賴於執行系統的被動告警和主動探測，而CI則使用單元和整合測試來檢測軟體在部署前的變化。在這兩個領域之間進行比較，可以讓我們把一個領域的知識應用到另一個領域。

Both CI and alerting serve the same overall purpose in the developer workflow—to identify problems as quickly as reasonably possible. CI emphasizes the early side of the developer workflow, and catches problems by surfacing test failures. Alerting focuses on the late end of the same workflow and catches problems by monitoring metrics and reporting when they exceed some threshold. Both are forms of “identify problems automatically, as soon as possible.”

CI和告警在開發者工作流程中的總體目的是一樣的--儘可能快地發現問題。CI強調開發者工作流程的早期階段，並透過顯示測試失敗來捕獲問題。告警側重於同一工作流程的後期，透過監測指標並在指標超過某個閾值時報告來捕捉問題。兩者都是 "自動、儘快地識別問題"的形式。

A well-managed alerting system helps to ensure that your Service-Level Objectives (SLOs) are being met. A good CI system helps to ensure that your build is in good shape—the code compiles, tests pass, and you could deploy a new release if you needed to. Best-practice policies in both spaces focus a lot on ideas of fidelity and actionable alerting: tests should fail only when the important underlying invariant is violated, rather than because the test is brittle or flaky. A flaky test that fails every few CI runs is just as much of a problem as a spurious alert going off every few minutes and generating a page for the on-call. If it isn’t actionable, it shouldn’t be alerting. If it isn’t actually violating the invariants of the SUT, it shouldn’t be a test failure.

一個管理良好的告警系統有助於確保你的服務水平目標（SLO）得到滿足。一個好的CI系統有助於確保你的建構處於良好狀態--程式碼編譯，測試透過，如果需要的話，你可以部署一個新版本。這兩個領域的最佳實踐策略都非常注重模擬度和可操作的告警：測試應該只在重要的基礎不變因素被違反時才失敗，而不是因為測試很脆弱或不穩定。一個脆弱的測試，每執行幾次CI就會失敗，就像一個虛假的警報每隔幾分鐘就會響起，並為值班人員產生一個頁面一樣，是一個問題。如果它不具有可操作性，就不應該發出警報。如果它實際上沒有違反SUT的不變性，就不應該是測試失敗。

CI and alerting share an underlying conceptual framework. For instance, there’s a similar relationship between localized signals (unit tests, monitoring of isolated statistics/cause-based alerting) and cross-dependency signals (integration and release tests, black-box probing). The highest fidelity indicators of whether an aggregate system is working are the end-to-end signals, but we pay for that fidelity in flakiness, increasing resource costs, and difficulty in debugging root causes.

CI和告警共享一個基本的概念框架。例如，在區域性訊號（單元測試、獨立統計監測/基於原因的警報）和交叉依賴訊號（整合和發佈測試、黑盒探測）之間存在類似的關係。衡量一個整體系統是否工作的最高模擬度指標是端到端的訊號，但我們要為這種模擬度付出代價，即鬆散性、不斷增加的資源成本和除錯根源的難度。

Similarly, we see an underlying connection in the failure modes for both domains. Brittle cause-based alerts fire based on crossing an arbitrary threshold (say, retries in the past hour), without there necessarily being a fundamental connection between that threshold and system health as seen by an end user. Brittle tests fail when an arbitrary test requirement or invariant is violated, without there necessarily being a fundamental connection between that invariant and the correctness of the software being tested. In most cases these are easy to write, and potentially helpful in debugging a larger issue. In both cases they are rough proxies for overall health/correctness, failing to capture the holistic behavior. If you don’t have an easy end-to-end probe, but you do make it easy to collect some aggregate statistics, teams will write threshold alerts based on arbitrary statistics. If you don’t have a high-level way to say, “Fail the test if the decoded image isn’t roughly the same as this decoded image,” teams will instead build tests that assert that the byte streams are identical.

同樣，我們在這兩個領域的故障模式中看到了一種潛在的聯絡。脆弱的基於原因的告警基於超過任意閾值（例如，過去一小時內的重試）而啟動，而該閾值與終端使用者看到的系統健康狀況之間不一定有根本聯絡。當一個任意的測試要求或不變數被違反時，脆性測試就會失敗，而不一定在該不變數和被測軟體的正確性之間有根本的聯絡。在大多數情況下，這些測試很容易寫，並有可能有助於除錯更大的問題。在這兩種情況下，它們都是整體健康/正確性的粗略代理，無法捕獲整體行為。如果你沒有一個簡單的端到端探針，但你確實可以輕鬆地收集一些聚合統計資訊，那麼團隊將基於任意統計資訊編寫閾值警報。如果你沒有一個高層次的方法說："如果解碼後的影象與這個解碼後的影象不大致相同，則測試失敗"，團隊就會建立測試，斷言位元組流是相同的。

Cause-based alerts and brittle tests can still have value; they just aren’t the ideal way to identify potential problems in an alerting scenario. In the event of an actual failure, having more debug detail available can be useful. When SREs are debugging an outage, it can be useful to have information of the form, “An hour ago users, started experiencing more failed requests. Around the same, time the number of retries started ticking up. Let’s start investigating there.” Similarly, brittle tests can still provide extra debugging information: “The image rendering pipeline started spitting out garbage. One of the unit tests suggests that we’re getting different bytes back from the JPEG compressor. Let’s start investigating there.”

基於原因的告警和脆性測試仍然有價值；它們只是在告警場景中不是識別潛在問題的理想方式。在實際發生故障的情況下，有更多的除錯細節可以使用。當SRE正在除錯一個故障時，有這樣的資訊是很有用的："一小時前，使用者開始遇到更多的失敗請求。大約在同一時間，重試的數量開始上升。讓我們開始調查。" 同樣地，脆弱的測試仍然可以提供額外的除錯資訊。"影象渲染管道開始吐出垃圾。其中一個單元測試表明，我們從JPEG壓縮器那裡得到了不同的位元組。讓我們開始調查吧。"

Although monitoring and alerting are considered a part of the SRE/production management domain, where the insight of “Error Budgets” is well understood,[^9] CI comes from a perspective that still tends to be focused on absolutes. Framing CI as the “left shift” of alerting starts to suggest ways to reason about those policies and propose better best practices:

•   Having a 100% green rate on CI, just like having 100% uptime for a production service, is awfully expensive. If that is *actually* your goal, one of the biggest problems is going to be a race condition between testing and submission.

•   Treating every alert as an equal cause for alarm is not generally the correct approach. If an alert fires in production but the service isn’t actually impacted, silencing the alert is the correct choice. The same is true for test failures: until our CI systems learn how to say, “This test is known to be failing for irrelevant reasons,” we should probably be more liberal in accepting changes that disable a failed test. Not all test failures are indicative of upcoming production issues.

•   Policies that say, “Nobody can commit if our latest CI results aren’t green” are probably misguided. If CI reports an issue, such failures should definitely be *investigated* before letting people commit or compound the issue. But if the root cause is well understood and clearly would not affect production, blocking commits is unreasonable.

儘管監控和告警被認為是SRE/生產管理領域的一部分，其中 "錯誤成本 "的洞察力被很好地理解，CI來自一個仍然傾向於關注絕對性的視角。將CI定義為告警的 "左移"，開始建議如何推理這些策略並提出更好的最佳實踐：

- 在CI上實現100%的綠色率，就像在生產服務中實現100%的正常執行時間一樣，是非常昂貴的。如果這確實是你的目標，那麼最大的問題之一就是測試和提交之間的競爭條件。

- 把每一個告警都當作一個相同原因來處理，一般來說不是正確的方法。如果一個告警在生產中被觸發，但服務實際上並沒有受到影響，讓告警沉默是正確的選擇。對於測試失敗也是如此：在我們的CI系統學會如何說“已知此測試因無關原因而失敗”之前，我們可能應該更自由地接受禁用失敗測試的更改。並非所有測試失敗都表明即將出現生產問題。

- 那些說 "如果我們最新的CI結果不是綠色的，任何人都不能提交 "的策略可能是錯誤的。如果 CI 報告了一個問題，在讓人們提交或使問題複雜化之前，肯定要對這種失敗進行調查。但如果根本原因已被充分理解，並且顯然不會影響生產，那麼阻止提交是不合理的。

This “CI is alerting” insight is new, and we’re still figuring out how to fully draw parallels. Given the higher stakes involved, it’s unsurprising that SRE has put a lot of thought into best practices surrounding monitoring and alerting, whereas CI has been viewed as more of a luxury feature.[^10] For the next few years, the task in software engineering will be to see where existing SRE practice can be reconceptualized in a CI context to help reformulate the testing and CI landscape—and perhaps where best practices in testing can help clarify goals and policies on monitoring and alerting.

這種 "CI就是警報 "的見解是新的，我們仍在摸索如何充分地得出相似之處。鑑於所涉及的風險較高，SRE對圍繞監控和警報的最佳實踐進行了大量的思考，而CI則被視為一種奢侈的功能，這一點並不奇怪。在未來幾年，軟體工程的任務將是看看現有的SRE實踐可以在CI背景下重新概念化，以幫助重新制定測試和CI景觀，也許測試的最佳實踐可以幫助澄清監控和警報的目標和策略。

----

> 9	Aiming for 100% uptime is the wrong target. Pick something like 99.9% or 99.999% as a business or product trade-off, define and monitor your actual uptime, and use that “budget” as an input to how aggressively you’re willing to push risky releases./
> 9 以100%的正常執行時間為目標是錯誤的。選擇像99.9%或99.999%這樣的目標作為業務或產品的權衡，定義並監控你的實際正常執行時間，並使用該 "成本預算 "作為你願意多積極地推動風險發佈的輸入。
>
> 10	We believe CI is actually critical to the software engineering ecosystem: a must-have, not a luxury. But that is not universally understood yet./
> 10 我們相信CI實際上對軟體工程生態系統至關重要：它是必需品，而不是奢侈品。但這一點尚未得到普遍理解。


### CI Challenges

We’ve discussed some of the established best practices in CI and have introduced some of the challenges involved, such as the potential disruption to engineer productivity of unstable, slow, conflicting, or simply too many tests at presubmit. Some common additional challenges when implementing CI include the following:

- *Presubmit optimization*  
    Including *which* tests to run at presubmit time given the potential issues we’ve already described, and *how* to run them.

- *Culprit finding* and *failure isolation*  
    Which code or other change caused the problem, and which system did it happen in? “Integrating upstream microservices” is one approach to failure isolation in a distributed architecture, when you want to figure out whether a problem originated in your own servers or a backend. In this approach, you stage combinations of your stable servers along with upstream microservices’ new servers. (Thus, you are integrating the microservices’ latest changes into your testing.) This approach can be particularly challenging due to version skew: not only are these environments often incompatible, but you’re also likely to encounter false positives—problems that occur in a particular staged combination that wouldn’t actually be spotted in production.

- *Resource constraints*  
    Tests need resources to run, and large tests can be very expensive. In addition, the cost for the infrastructure for inserting automated testing throughout the process can be considerable.

我們已經討論了CI的一些已確認的最佳實踐，並介紹了其中的一些挑戰，例如不穩定的、緩慢的、衝突的或僅僅是在預提交時太多的測試對工程師生產力的潛在干擾。實施CI時，一些常見的額外挑戰包括以下內容：

- *提交前優化*  
    包括考慮到我們已經描述過的潛在問題，在提交前執行哪些測試，以及如何執行它們。

- *找出罪魁禍首*和*故障隔離*  
    哪段程式碼或其他變化導致了問題，它發生在哪個系統中？"整合上游微服務 "是分散式架構中故障隔離的一種方法，當你想弄清楚問題是源於你自己的伺服器還是後端。在這種方法中，你把你的穩定伺服器與上游微服務的新伺服器組合在一起。(因此，你將微服務的最新變化整合到你的測試中）。由於版本偏差，這種方法可能特別具有挑戰性：不僅這些環境經常不相容，而且你還可能遇到假陽性--在某個特定的階段性組合中出現的問題，實際上在生產中不會被發現。

- *資源限制*  
    測試需要資源來執行，而大型測試可能非常昂貴。此外，在整個過程中插入自動化測試的基礎設施的成本可能是相當大的。

There’s also the challenge of *failure management—*what to do when tests fail. Although smaller problems can usually be fixed quickly, many of our teams find that it’s extremely difficult to have a consistently green test suite when large end-to-end tests are involved. They inherently become broken or flaky and are difficult to debug; there needs to be a mechanism to temporarily disable and keep track of them so that the release can go on. A common technique at Google is to use bug “hotlists” filed by an on-call or release engineer and triaged to the appropriate team. Even better is when these bugs can be automatically generated and filed—some of our larger products, like Google Web Server (GWS) and Google Assistant, do this. These hotlists should be curated to make sure any release-blocking bugs are fixed immediately. Nonrelease blockers should be fixed, too; they are less urgent, but should also be prioritized so the test suite remains useful and is not simply a growing pile of disabled, old tests. Often, the problems caught by end-to-end test failures are actually with tests rather than code.

還有一個挑戰是*失敗管理*--當測試失敗時該怎麼做。儘管較小的問題通常可以很快得到解決，但我們的許多團隊發現，當涉及到大型的端到端測試時，要有一個持續的綠色測試套件是非常困難的。它們本來就會出現故障或不穩定，而且難以除錯；需要有一種機制來暫時禁用並追蹤它們，以便發佈工作能夠繼續進行。在谷歌，一種常見的技術是使用由值班或發佈工程師提交的bug "熱名單"，並將其分發給相應的團隊。如果這些bug能夠自動產生並歸檔，那就更好了--我們的一些大型產品，如谷歌網路伺服器（GWS）和谷歌助手，就能做到這一點。應對這些熱名單進行整理，以確保立即修復所有阻止發佈的bug。非發佈障礙也應該被修復；它們不那麼緊急，但也應該被優先處理，這樣測試套件才會保持有用，而不僅僅是一堆越來越多的失效的舊測試。通常，由端到端測試失敗引起的問題實際上是測試問題，而不是程式碼問題。

Flaky tests pose another problem to this process. They erode confidence similar to a broken test, but finding a change to roll back is often more difficult because the failure won’t happen all the time. Some teams rely on a tool to remove such flaky tests from presubmit temporarily while the flakiness is investigated and fixed. This keeps confidence high while allowing for more time to fix the problem.

不穩定測試給這個過程帶來了另一個問題。它們會侵蝕信心，就像一次失敗的測試一樣，但找到一個可以回滾的變化往往更困難，因為失敗不會一直髮生。一些團隊依靠一種工具，在調查和修復不穩定的測試時，暫時從預提交中刪除這種不穩定測試。這樣可以保持較高的信心，同時允許有更多的時間來修復這個問題。

*Test instability* is another significant challenge that we’ve already looked at in the context of presubmits. One tactic for dealing with this is to allow multiple attempts of the test to run. This is a common test configuration setting that teams use. Also, within test code, retries can be introduced at various points of specificity.

*測試的不穩定性*是另一個重大挑戰，我們已經在預提交的背景下看過了。處理這個問題的一個策略是允許測試的多次嘗試執行。這是團隊使用的常見測試配置設定。另外，在測試程式碼中，可以在不同的特定點引入重試。

Another approach that helps with test instability (and other CI challenges) is hermetic testing, which we’ll look at in the next section.

另一種有助於解決測試不穩定性（和其他CI挑戰）的方法是封閉測試，我們將在下一節中討論。

### Hermetic Testing  封閉測試

Because talking to a live backend is unreliable, we often use [hermetic backends ](https://oreil.ly/-PbRM)for larger-scoped tests. This is particularly useful when we want to run these tests on presubmit, when stability is of utmost importance. In [Chapter 11](#_bookmark838), we introduced the concept of hermetic tests:

*Hermetic tests*: tests run against a test environment (i.e., application servers and resources) that is entirely self-contained (i.e., no external dependencies like production backends).

因為與實時後端互動是不可靠的，我們經常使用[封閉後端](https://oreil.ly/-PbRM)進行較大範圍的測試。當我們想在提交前執行這些測試時，這是特別有用的，因為此時穩定性是最重要的。在第11章中，我們介紹了封閉測試的概念。

​	*封閉測試*：針對測試環境（即應用伺服器和資源）執行的測試，是完全自成一體的（即沒有像生產後端那樣的外部依賴）。

Hermetic tests have two important properties: greater determinism (i.e., stability) and isolation. Hermetic servers are still prone to some sources of nondeterminism, like system time, random number generation, and race conditions. But, what goes into the test doesn’t change based on outside dependencies, so when you run a test twice with the same application and test code, you should get the same results. If a hermetic test fails, you know that it’s due to a change in your application code or tests (with a minor caveat: they can also fail due to a restructuring of your hermetic test environment, but this should not change very often). For this reason, when CI systems rerun tests hours or days later to provide additional signals, hermeticity makes test failures easier to narrow down.

封閉測試有兩個重要的特性：更高的確定性（即穩定性）和隔離性。封閉式伺服器仍然容易受到一些非確定性來源的影響，如系統時間、隨機數產生和競態條件。但是，進入測試的內容不會因為外部的依賴關係而改變，所以當你用相同的應用程式和測試程式碼執行兩次測試時，你應該得到相同的結果。如果一個封閉測試失敗了，你就知道是由於你的應用程式程式碼或測試的變化造成的（有一個小的注意事項：它們也可能由於你的封閉測試環境的重組而失敗，但這不應該經常更變）。出於這個原因，當CI系統在幾小時或幾天後重新執行測試以提供額外的訊號時，封閉性使測試失敗更容易縮小範圍。

The other important property, isolation, means that problems in production should not affect these tests. We generally run these tests all on the same machine as well, so we don’t have to worry about network connectivity issues. The reverse also holds: problems caused by running hermetic tests should not affect production.

另一個重要特性，隔離，意味著生產環境中的問題不應該影響這些測試。我們通常也在同一臺機器上執行這些測試，因此我們不必擔心網路連線問題。反之亦然：執行封閉測試引起的問題不應影響生產環境。

Hermetic test success should not depend on the user running the test. This allows people to reproduce tests run by the CI system and allows people (e.g., library developers) to run tests owned by other teams.

封閉測試的成功不應取決於執行測試的使用者。這允許人們複製CI系統執行的測試，並允許人們（例如，函式庫的開發者）執行其他團隊擁有的測試。

One type of hermetic backend is a fake. As discussed in [Chapter 13](#_bookmark1056), these can be cheaper than running a real backend, but they take work to maintain and have limited fidelity.

一種封閉式的後端是模擬的。正如在第13章中所討論的，這些可能比執行一個真正的後端更廉價，但它們需要花費精力去維護，而且模擬度有限。

The cleanest option to achieve a presubmit-worthy integration test is with a fully hermetic setup—that is, starting up the entire stack sandboxed[^11]—and Google provides out-of-the-box sandbox configurations for popular components, like databases, to make it easier. This is more feasible for smaller applications with fewer components, but there are exceptions at Google, even one (by DisplayAds) that starts about four hundred servers from scratch on every presubmit as well as continuously on post- submit. Since the time that system was created, though, record/replay has emerged as a more popular paradigm for larger systems and tends to be cheaper than starting up a large sandboxed stack.

實現具有預提交價值的整合測試的最乾淨的選擇是使用一個完全精細的設定--即啟動整個堆疊沙盒--谷歌為流行元件（如資料庫）提供開箱即用的沙盒配置，以使其更簡單。這對於元件較少的小型應用程式更為可行，但谷歌也有例外，即使是一個（由DisplayAds提供）在每次提交前以及提交後從零開始啟動大約400臺伺服器的應用程式。但是，自建立該系統以來，錄製/重播已成為大型系統的一種更受歡迎的範例，並且往往比啟動大型沙盒堆疊更便宜。

Record/replay (see [Chapter 14](#_bookmark1181)) systems record live backend responses, cache them, and replay them in a hermetic test environment. Record/replay is a powerful tool for reducing test instability, but one downside is that it leads to brittle tests: it’s difficult to strike a balance between the following:

*False positives*

​	The test passes when it probably shouldn’t have because we are hitting the cache too much and missing problems that would surface when capturing a new response.

*False negatives*

​	The test fails when it probably shouldn’t have because we are hitting the cache too little. This requires responses to be updated, which can take a long time and lead to test failures that must be fixed, many of which might not be actual problems. This process is often submit-blocking, which is not ideal.

記錄/重放（見第14章）系統記錄即時的後端響應，快取它們，並在一個封閉的測試環境中重放它們。記錄/重放是一個強大的工具，可以減少測試的不穩定性，但一個缺點是它會導致測試變脆弱：很難在以下方面取得平衡：

*假陽性*
​	測試在不應該透過的情況下通過了，因為我們對快取的訪問太多，並且遺漏了捕獲新響應時可能出現的問題。

*錯誤的否定*
​	測試在不應該透過的情況下失敗了，因為我們對緩衝區的命中太少。這需要更新響應，這可能需要很長時間，並導致必須修復的測試失敗，其中許多可能不是實際問題。這個過程通常是提交阻塞，這並不理想。

Ideally, a record/replay system should detect only problematic changes and cache- miss only when a request has changed in a meaningful way. In the event that that change causes a problem, the code change author would rerun the test with an updated response, see that the test is still failing, and thereby be alerted to the problem. In practice, knowing when a request has changed in a meaningful way can be incredibly difficult in a large and ever-changing system.

理想情況下，記錄/重放系統應該只檢測有問題的更改，並且只有在請求以有意義的方式更改時才檢測快取未命中。如果該更改導致問題，程式碼修改者會用更新的響應重新執行測試，檢視測試是否仍然失敗，並因此收到問題警報。在實踐中，在一個大型且不斷變化的系統中，知道請求何時以有意義的方式發生了更改可能非常困難。

> [^11]: In practice, it’s often difficult to make a completely sandboxed test environment, but the desired stability can be achieved by minimizing outside dependencies.
> 11 在實踐中，通常很難做出一個完全沙盒化的測試環境，但可以透過儘量減少外部的依賴性來實現所需的穩定性。

-----

#### The Hermetic Google Assistant 隱祕的谷歌助手

Google Assistant provides a framework for engineers to run end-to-end tests, including a test fixture with functionality for setting up queries, specifying whether to simulate on a phone or a smart home device, and validating responses throughout an exchange with Google Assistant.

谷歌助手為工程師提供了一個執行端到端測試的框架，包括一個具有設定查詢功能的測試套件，指定是否在手機或智慧家居裝置上進行模擬，並在與谷歌助手的整個互動中驗證響應。

One of its greatest success stories was making its test suite fully hermetic on presubmit. When the team previously used to run nonhermetic tests on presubmit, the tests would routinely fail. In some days, the team would see more than 50 code changes bypass and ignore the test results. In moving presubmit to hermetic, the team cut the runtime by a factor of 14, with virtually no flakiness. It still sees failures, but those failures tend to be fairly easy to find and roll back.

其最大的成功故事之一是使其測試套件在提交前完全密封。當該團隊以前在提交前執行非封閉測試時，測試經常會失敗。在某些日子裡，團隊會看到超過50個程式碼更改繞過並忽略測試結果。在將預提交轉為封閉的過程中，該團隊將執行時間縮短了14倍，而且幾乎沒有任何閃失。它仍然會出現故障，但這些故障往往是相當容易發現和回滾的。

Now that nonhermetic tests have been pushed to post-submit, it results in failures accumulating there instead. Debugging failing end-to-end tests is still difficult, and some teams don’t have time to even try, so they just disable them. That’s better than having it stop all development for everyone, but it can result in production failures.

現在，非封閉測試已經被推到提交後，結果反而導致失敗在那裡累積。除錯失敗的端到端測試仍然很困難，一些團隊甚至沒有時間嘗試，所以他們只是禁用它們。這比讓它停止所有人的開發要好，但它可能導致生產失敗。

One of the team’s current challenges is to continue to fine-tuning its caching mechanisms so that presubmit can catch more types of issues that have been discovered only post-submit in the past, without introducing too much brittleness.

該團隊目前的挑戰之一是繼續微調其快取機制，以便預提交可以捕捉到更多過去只在提交後發現的問題型別，同時不引入過多的脆弱性。

Another is how to do presubmit testing for the decentralized Assistant given that components are shifting into their own microservices. Because the Assistant has a large and complex stack, the cost of running a hermetic stack on presubmit, in terms of engineering work, coordination, and resources, would be very high.

另一個問題是，鑑於元件正在轉移到自己的微服務中，如何為分散的助手做預提交測試。因為助手有一個龐大而複雜的堆疊，在預提交上執行一個封閉的堆疊，在工程工作、協調和資源方面的成本會非常高。

Finally, the team is taking advantage of this decentralization in a clever new post- submit failure-isolation strategy. For each of the *N* microservices within the Assistant, the team will run a post-submit environment containing the microservice built at head, along with production (or close to it) versions of the other *N* – 1 services, to isolate problems to the newly built server. This setup would normally be *O*(*N*2) cost to facilitate, but the team leverages a cool feature called *hotswapping* to cut this cost to *O*(*N*). Essentially, hotswapping allows a request to instruct a server to “swap” in the address of a backend to call instead of the usual one. So only *N* servers need to be run, one for each of the microservices cut at head—and they can reuse the same set of prod backends swapped in to each of these *N* “environments.”

最後，該團隊正在利用這種分散的優勢，採取一種巧妙的新的提交後故障隔離策略。對於助手中的N個微服務中的每一個，團隊將執行一個提交後的環境，其中包含在頭部建構的微服務，以及其他N-1個服務的生產（或接近生產）版本，以將問題隔離到新建構的伺服器。這種設定通常是O(N2)的成本，但該團隊利用了一個很酷的功能，稱為熱交換，將這一成本削減到O(N)。從本質上講，"熱交換 "允許一個請求指示伺服器 "交換 "一個後端地址來呼叫，而不是通常的一個。因此，只需要執行N個伺服器，每個微服務都有一個，而且它們可以重複使用同一組被交換到這N個 "環境 "中的生產環境後端。

-----

As we’ve seen in this section, hermetic testing can both reduce instability in larger- scoped tests and help isolate failures—addressing two of the significant CI challenges we identified in the previous section. However, hermetic backends can also be more expensive because they use more resources and are slower to set up. Many teams use combinations of hermetic and live backends in their test environments.

正如我們在本節中所看到的，封閉測試既可以減少大範圍測試中的不穩定性，也可以幫助隔離故障，解決我們在上一節中確定的兩個重大CI挑戰。然而，封閉式後端也可能更昂貴，因為它們使用更多的資源，並且設定速度較慢。許多團隊在他們的測試環境中使用密封和活動後端的組合。

## CI at Google 谷歌的CI

Now let’s look in more detail at how CI is implemented at Google. First, we’ll look at our global continuous build, TAP, used by the vast majority of teams at Google, and how it enables some of the practices and addresses some of the challenges that we looked at in the previous section. We’ll also look at one application, Google Takeout, and how a CI transformation helped it scale both as a platform and as a service.

現在讓我們更詳細地看看CI在谷歌是如何實施的。首先，我們將瞭解谷歌絕大多數團隊使用的全球持續建構TAP，以及它是如何實現一些實踐和解決我們在上一節中看到的一些挑戰的。我們還將介紹一個應用程式Google Takeout，以及CI轉換如何幫助其作為平臺和服務進行擴充套件。

-----

### TAP: Google’s Global Continuous Build 谷歌的全球持續建構

Adam Bender 亞當-本德 

We run a massive continuous build, called the Test Automation Platform (TAP), of our entire codebase. It is responsible for running the majority of our automated tests. As a direct consequence of our use of a monorepo, TAP is the gateway for almost all changes at Google. Every day it is responsible for handling more than 50,000 unique changes *and* running more than four billion individual test cases.

我們在整個程式碼函式庫中執行一個大規模的持續建構，稱為測試自動化平臺（TAP）。它負責執行我們大部分的自動化測試。由於我們使用的是monorepo，TAP是谷歌幾乎所有變化的門戶。每天，它負責處理超過50,000個獨特的變化，執行超過40億個單獨的測試用例。

TAP is the beating heart of Google’s development infrastructure. Conceptually, the process is very simple. When an engineer attempts to submit code, TAP runs the associated tests and reports success or failure. If the tests pass, the change is allowed into the codebase.

TAP是谷歌發展基礎設施的核心。從概念上講，這個過程非常簡單。當工程師試圖提交程式碼時，TAP將執行相關測試並報告成功或失敗。如果測試透過，則允許更改進入程式碼函式庫。

#### Presubmit optimization 預提交優化

To catch issues quickly and consistently, it is important to ensure that tests are run against every change. Without a CB, running tests is usually left to individual engineer discretion, and that often leads to a few motivated engineers trying to run all tests and keep up with the failures.

為了快速和持續地發現問題，必須確保對每一個變化都進行測試。如果沒有CB，執行測試通常是由個別工程師決定的，這往往會導致一些有積極性的工程師試圖執行所有的測試並跟進故障。

As discussed earlier, waiting a long time to run every test on presubmit can be severely disruptive, in some cases taking hours. To minimize the time spent waiting, Google’s CB approach allows potentially breaking changes to land in the repository (remember that they become immediately visible to the rest of the company!). All we ask is for each team to create a fast subset of tests, often a project’s unit tests, that can be run before a change is submitted (usually before it is sent for code review)—the presubmit. Empirically, a change that passes the presubmit has a very high likelihood (95%+) of passing the rest of the tests, and we optimistically allow it to be integrated so that other engineers can then begin to use it.

如前所述，等待很長時間來執行預提交的每個測試可能會造成嚴重破壞，在某些情況下需要數小時。為了最大限度地減少等待時間，谷歌的CB方法允許潛在的破壞性更改提交到儲存函式庫中（請記住，這些更改會立即被公司其他人看到！）。我們只要求每個團隊建立一個快速的測試子集，通常是一個專案的單元測試，可以在提交更改之前（通常是在傳送更改進行程式碼審查之前）執行這些測試。根據經驗，透過預提交的變更透過其餘測試的可能性非常高（95%+），我們樂觀地允許將其整合，以便其他工程師可以開始使用它。

After a change has been submitted, we use TAP to asynchronously run all potentially affected tests, including larger and slower tests.

提交更改後，我們使用TAP非同步執行所有可能受影響的測試，包括較大和較慢的測試。

When a change causes a test to fail in TAP, it is imperative that the change be fixed quickly to prevent blocking other engineers. We have established a cultural norm that strongly discourages committing any new work on top of known failing tests, though flaky tests make this difficult. Thus, when a change is committed that breaks a team’s build in TAP, that change may prevent the team from making forward progress or building a new release. As a result, dealing with breakages quickly is imperative.

當變更導致TAP測試失敗時，必須迅速修復變更，以防止阻塞其他工程師。我們已經建立了一種文化規範，強烈反對在已知失敗測試的基礎上進行任何新的工作，儘管不穩定測試會讓這變得困難。因此，當提交的變更打破了團隊的內建TAP時，該變更可能會阻止團隊向前推進或建構新版本。因此，快速處理故障勢在必行。

To deal with such breakages, each team has a “Build Cop.” The Build Cop’s responsibility is keeping all the tests passing in their particular project, regardless of who breaks them. When a Build Cop is notified of a failing test in their project, they drop whatever they are doing and fix the build. This is usually by identifying the offending change and determining whether it needs to be rolled back (the preferred solution) or can be fixed going forward (a riskier proposition).

為了處理這種破壞，每個團隊都有一個 "Build Cop"。Build Cop的責任是保持他們特定專案的所有測試透過，無論誰破壞了它們。當Build Cop被告知他們的專案中有一個失敗的測試時，他們會放下手中的工作，修復建構。這通常是透過識別違規的變化，並確定它是否需要回滾（首選解決方案）或可以繼續修復（風險較大）。

In practice, the trade-off of allowing changes to be committed before verifying all tests has really paid off; the average wait time to submit a change is around 11 minutes, often run in the background. Coupled with the discipline of the Build Cop, we are able to efficiently detect and address breakages detected by longer running tests with a minimal amount of disruption.

在實踐中，允許在驗證所有測試之前提交更改的折衷方案已經真正得到了回報；提交更改的平均等待時間約為11分鐘，通常在後臺執行。再加上Build Cop的原則，我們能夠以最小的中斷量有效地檢測和解決執行時間較長的測試檢測到的故障。

#### Culprit finding發現罪魁禍首

One of the problems we face with large test suites at Google is finding the specific change that broke a test. Conceptually, this should be really easy: grab a change, run the tests, if any tests fail, mark the change as bad. Unfortunately, due to a prevalence of flakes and the occasional issues with the testing infrastructure itself, having confidence that a failure is real isn’t easy. To make matters more complicated, TAP must evaluate so many changes a day (more than one a second) that it can no longer run every test on every change. Instead, it falls back to batching related changes together, which reduces the total number of unique tests to be run. Although this approach can make it faster to run tests, it can obscure which change in the batch caused a test to break.

谷歌大型測試套件面臨的一個問題是找到破壞測試的具體變化。從概念上講，這應該很容易：抓取一個變更，執行測試，如果任何測試失敗，將變更標記為壞的。不幸的是，由於片斷的流行以及測試基礎設施本身偶爾出現的問題，要確信失敗是真實的並不容易。更加複雜的是，TAP必須每天評估如此多的變化（一秒鐘超過一個），以至於它不能再對每個變化執行每個測試。取而代之的是，它退回到批處理相關的更改，這減少了要執行的獨特測試的總數。儘管這種方法可以加快執行測試的速度，但它可以掩蓋批處理中導致測試中斷的更改。

To speed up failure identification, we use two different approaches. First, TAP automatically splits a failing batch up into individual changes and reruns the tests against each change in isolation. This process can sometimes take a while to converge on a failure, so in addition, we have created culprit finding tools that an individual developer can use to binary search through a batch of changes and identify which one is the likely culprit.

為了加快故障識別，我們使用了兩種不同的方法。首先，TAP自動將失敗的批次拆分為單獨的更改，並針對每個更改單獨重新執行測試。這個過程有時需要一段時間才能收斂到失敗，因此，我們還建立了罪魁禍首查詢工具，每個開發人員可以使用這些工具透過一批更改進行二進位制搜尋，並確定哪一個是可能的罪魁禍首。

#### Failure management 故障管理

After a breaking change has been isolated, it is important to fix it as quickly as possible. The presence of failing tests can quickly begin to erode confidence in the test suite. As mentioned previously, fixing a broken build is the responsibility of the Build Cop. The most effective tool the Build Cop has is the *rollback*.

在隔離破壞性變更後，儘快修復該變更非常重要。失敗測試的存在可能會很快開始侵蝕測試套件的信心。如前所述，修復損壞的建構是Build Cop的責任。Build Cop最有效的工具是*回滾*。

Rolling a change back is often the fastest and safest route to fix a build because it quickly restores the system to a known good state.[^12] In fact, TAP has recently been upgraded to automatically roll back changes when it has high confidence that they are the culprit.

回滾更改通常是修復產生的最快和最安全的方法，因為它可以快速將系統恢復到已知的良好狀態。事實上，TAP最近已升級為自動回滾更改，當它高度確信更改是罪魁禍首時。

Fast rollbacks work hand in hand with a test suite to ensure continued productivity. Tests give us confidence to change, rollbacks give us confidence to undo. Without tests, rollbacks can’t be done safely. Without rollbacks, broken tests can’t be fixed quickly, thereby reducing confidence in the system.

快速回滾與測試套件攜手並進，以確保持續的生產力。測試給了我們改變的信心，回滾給了我們撤銷的信心。沒有測試，回滾就不能安全進行。沒有回滾，破損的測試就不能被快速修復，從而降低了對系統的信心。

> 12 Any change to Google’s codebase can be rolled back with two clicks!
> 12 對谷歌程式碼函式庫的任何改動都可以透過兩次點選來回滾。

#### Resource constraints 資源限制

Although engineers can run tests locally, most test executions happen in a distributed build-and-test system called *Forge*. Forge allows engineers to run their builds and tests in our datacenters, which maximizes parallelism. At our scale, the resources required to run all tests executed on-demand by engineers and all tests being run as part of the CB process are enormous. Even given the amount of compute resources we have, systems like Forge and TAP are resource constrained. To work around these constraints, engineers working on TAP have come up with some clever ways to determine which tests should be run at which times to ensure that the minimal amount of resources are spent to validate a given change.

雖然工程師可以在本地執行測試，但大多數測試的執行是在一個叫做*Forge*的分散式建構和測試系統中進行。Forge允許工程師在我們的資料中心執行他們的建構和測試，這最大限度地提高了並行性。在我們的規模下，執行所有由工程師按需執行的測試以及作為CB流程一部分執行的所有測試所需的資源是巨大的。即使考慮到我們擁有的計算資源量，像Forge和TAP這樣的系統也受到資源限制。為了解決這些限制，在TAP上工作的工程師想出了一些聰明的方法來確定哪些測試應該在什麼時候執行，以確保花費最少的資源來驗證一個特定的變化。

The primary mechanism for determining which tests need to be run is an analysis of the downstream dependency graph for every change. Google’s distributed build tools, Forge and Blaze, maintain a near-real-time version of the global dependency graph and make it available to TAP. As a result, TAP can quickly determine which tests are downstream from any change and run the minimal set to be sure the change is safe.

確定需要執行哪些測試的主要機制是分析每個更改的下游依賴關係圖。谷歌的分散式建構工具Forge和Blaze維護了一個近乎即時的全球依賴關係圖版本，並可供使用者使用。因此，TAP可以快速確定任何更改的下游測試，並執行最小集以確保更改是安全的。

Another factor influencing the use of TAP is the speed of tests being run. TAP is often able to run changes with fewer tests sooner than those with more tests. This bias encourages engineers to write small, focused changes. The difference in waiting time between a change that triggers 100 tests and one that triggers 1,000 can be tens of minutes on a busy day. Engineers who want to spend less time waiting end up making smaller, targeted changes, which is a win for everyone.

影響TAP使用的另一個因素是測試執行的速度。TAP通常能夠以更少的測試比更多測試更快地執行更改。這種情況鼓勵工程師編寫小而集中的更改。在繁忙的一天中，觸發100個測試的更改和觸發1000個測試的更改之間的等待時間差異可能是幾十分鐘。希望花更少時間等待的工程師最終會做出更小的、有針對性的修改，這對所有人來說都是一種勝利。 

----

### CI Case Study: Google Takeout   CI案例研究：Google Takeout

Google Takeout started out as a data backup and download product in 2011. Its founders pioneered the idea of “data liberation”—that users should be able to easily take their data with them, in a usable format, wherever they go. They began by integrating Takeout with a handful of Google products themselves, producing archives of users’ photos, contact lists, and so on for download at their request. However, Takeout didn’t stay small for long, growing as both a platform and a service for a wide variety of Google products. As we’ll see, effective CI is central to keeping any large project healthy, but is especially critical when applications rapidly grow.

2011年，Google Takeout開始作為一種資料備份和下載產品。其創始人率先提出了“資料解放”的理念，即使用者無論走到哪裡，都應該能夠輕鬆地以可用的格式攜帶資料。他們首先將Takeout與少量谷歌產品整合在一起，製作使用者照片、聯絡人列表等檔案，以便在他們的要求下下載。然而，Takeout並沒有在很長一段時間內保持規模，它不僅是一個平臺，而且是一項針對各種谷歌產品的服務。正如我們將看到的，有效的CI對於保持任何大型專案的健康至關重要，但在應用程式快速增長時尤為關鍵。

#### Scenario #1: Continuously broken dev deploys 情景#1：持續中斷的開發部署

**Problem:** As Takeout gained a reputation as a powerful Google-wide data fetching, archiving, and download tool, other teams at the company began to turn to it, requesting APIs so that their own applications could provide backup and download functionality, too, including Google Drive (folder downloads are served by Takeout) and Gmail (for ZIP file previews). All in all, Takeout grew from being the backend for just the original Google Takeout product, to providing APIs for at least 10 other Google products, offering a wide range of functionality.

**問題:**隨著Takeout作為功能強大的Google範圍內的資料獲取、歸檔和下載工具而聲名鵲起，該公司的其他團隊開始轉向它，請求API以便他們自己的應用程式也可以提供備份和下載功能，包括Google Drive（資料夾下載由Takeout提供）和Gmail（用於ZIP檔案預覽）. 總之，Takeout從最初的Google Takeout產品的後端發展到為至少10種其他Google產品提供API，提供廣泛的功能。

The team decided to deploy each of the new APIs as a customized instance, using the same original Takeout binaries but configuring them to work a little differently. For example, the environment for Drive bulk downloads has the largest fleet, the most quota reserved for fetching files from the Drive API, and some custom authentication logic to allow non-signed-in users to download public folders.

團隊決定將每個新的API部署為一個訂製的實例，使用相同的原始Takeout二進位制檔案，但將它們配置成有點不同的工作方式。例如，用於Drive批量下載的環境擁有最大的叢集，為從Drive API獲取檔案保留了最多的配額，以及一些自訂的認證邏輯，允許未登入的使用者下載公共資料夾。

Before long, Takeout faced “flag issues.” Flags added for one of the instances would break the others, and their deployments would break when servers could not start up due to configuration incompatibilities. Beyond feature configuration, there was security and ACL configuration, too. For example, the consumer Drive download service should not have access to keys that encrypt enterprise Gmail exports. Configuration quickly became complicated and led to nearly nightly breakages.

不久，Takeout就面臨“標誌問題”。為其中一個實例新增的標誌將破壞其他實例，當伺服器由於配置不相容而無法啟動時，它們的部署將中斷。除了功能配置之外，還有安全性和ACL配置。例如，消費者驅動器下載服務不應訪問加密企業Gmail匯出的金鑰。配置很快變得複雜，幾乎每晚都會發生故障。

Some efforts were made to detangle and modularize configuration, but the bigger problem this exposed was that when a Takeout engineer wanted to make a code change, it was not practical to manually test that each server started up under each configuration. They didn’t find out about configuration failures until the next day’s deploy. There were unit tests that ran on presubmit and post-submit (by TAP), but those weren’t sufficient to catch these kinds of issues.

我們做了一些努力來分解和模組化配置，但這暴露出的更大的問題是，當Takeout工程師想要修改程式碼時，手動測試每臺伺服器是否在每種配置下啟動是不切實際的。他們在第二天的部署中才發現配置失敗的情況。有一些單元測試是在提交前和提交後執行的（透過TAP），但這些測試不足以捕獲此類別問題。

**What the team did.** The team created temporary, sandboxed mini-environments for each of these instances that ran on presubmit and tested that all servers were healthy on startup. Running the temporary environments on presubmit prevented 95% of broken servers from bad configuration and reduced nightly deployment failures by 50%.

**團隊所做的**。**團隊為每個實例建立了臨時的、沙盒式的迷你環境，在預提交時執行，並測試所有伺服器在啟動時是否健康。在提交前執行臨時環境可以防止95%的伺服器因配置不當而損壞，並將夜間部署失敗率降低了50%。

Although these new sandboxed presubmit tests dramatically reduced deployment failures, they didn’t remove them entirely. In particular, Takeout’s end-to-end tests would still frequently break the deploy, and these tests were difficult to run on presubmit (because they use test accounts, which still behave like real accounts in some respects and are subject to the same security and privacy safeguards). Redesigning them to be presubmit friendly would have been too big an undertaking.

儘管這些新的沙盒式預提交測試大大減少了部署失敗，但它們並沒有完全消除它們。特別是，Takeout的端到端測試仍然經常中斷部署，而且這些測試很難在預提交中執行（因為它們使用的是測試賬戶，在某些方面仍然與真實賬戶一樣，並受到同樣的安全和隱私保護）。重新設計它們以使其對預提交友好，將是一項巨大的工程。

If the team couldn’t run end-to-end tests in presubmit, when could it run them? It wanted to get end-to-end test results more quickly than the next day’s dev deploy and decided every two hours was a good starting point. But the team didn’t want to do a full dev deploy this often—this would incur overhead and disrupt long-running processes that engineers were testing in dev. Making a new shared test environment for these tests also seemed like too much overhead to provision resources for, plus culprit finding (i.e., finding the deployment that led to a failure) could involve some undesirable manual work.

如果團隊不能在預提交中執行端到端測試，那麼它什麼時候可以執行？它想比第二天的開發部署更快得到端到端的測試結果，並決定每兩小時一次是一個好的起點。但團隊並不想這麼頻繁地進行全面的開發部署--這將產生開銷，並擾亂工程師在開發中測試的長期執行的流程。為這些測試建立一個新的共享測試環境，似乎也需要太多的開銷來提供資源，再加上查詢問題（即找到導致失敗的部署）可能涉及一些不可預知的手動工作。

So, the team reused the sandboxed environments from presubmit, easily extending them to a new post-submit environment. Unlike presubmit, post-submit was compliant with security safeguards to use the test accounts (for one, because the code has been approved), so the end-to-end tests could be run there. The post-submit CI runs every two hours, grabbing the latest code and configuration from green head, creates an RC, and runs the same end-to-end test suite against it that is already run in dev.

因此，該團隊重新使用了預提交的沙盒環境，輕鬆地將它們擴充套件到新的後提交環境。與預提交不同，後提交符合安全保障措施，可以使用測試賬戶（其一，因為程式碼已經被批准），所以端到端的測試可以在那裡執行。提交後的CI每兩小時執行一次，從綠頭抓取最新的程式碼和配置，建立一個RC，並針對它執行已經在開發中執行的相同的端到端測試套件。

**Lesson learned.** Faster feedback loops prevent problems in dev deploys:

- Moving tests for different Takeout products from “after nightly deploy” to presubmit prevented 95% of broken servers from bad configuration and reduced nightly deployment failures by 50%.

- Though end-to-end tests couldn’t be moved all the way to presubmit, they were still moved from “after nightly deploy” to “post-submit within two hours.” This effectively cut the “culprit set” by 12 times.

**經驗教訓。**更快的反饋迴圈防止了開發部署中的問題：

- 將不同Takeout產品的測試從 "夜間部署後 "轉移到預提交，可以防止95%的伺服器因配置不良而損壞，並將夜間部署的失敗率降低50%。

- 儘管端到端測試不能全部轉移到預提交，但它們仍然從 "夜間部署後 "轉移到 "兩小時內提交後"。這有效地將 "罪魁禍首集 "減少了12倍。

#### Scenario #2: Indecipherable test logs 場景2：無法識別的測試日誌

**Problem:** As Takeout incorporated more Google products, it grew into a mature platform that allowed product teams to insert plug-ins, with product-specific data- fetching code, directly into Takeout’s binary. For example, the Google Photos plug-in knows how to fetch photos, album metadata, and the like. Takeout expanded from its original “handful” of products to now integrate with more than *90*.

**問題：**隨著Takeout整合了更多的谷歌產品，它已經發展成為一個成熟的平臺，允許產品團隊直接在Takeout的二進位制檔案中插入外掛，其中包含產品特定的資料獲取程式碼。例如，谷歌照片外掛知道如何獲取照片、相簿元資料等。Takeout從最初的 "少數 "產品擴充套件到現在與超過*90個*的產品整合。

Takeout’s end-to-end tests dumped its failures to a log, and this approach didn’t scale to 90 product plug-ins. As more products integrated, more failures were introduced. Even though the team was running the tests earlier and more often with the addition of the post-submit CI, multiple failures would still pile up inside and were easy to miss. Going through these logs became a frustrating time sink, and the tests were almost always failing.

Takeout的端到端測試將其故障轉儲到日誌中，這種方法不能擴充套件到90個產品外掛。隨著更多產品的整合，更多的故障被引入。儘管團隊在提交後的CI中更早更頻繁地執行測試，但多個故障還是會堆積在裡面，很容易被忽略。翻閱這些日誌成了一個令人沮喪的時間消耗，而且測試幾乎總是失敗。

**What the team did.** The team refactored the tests into a dynamic, configuration-based suite (using a [parameterized test runner](https://oreil.ly/UxkHk)) that reported results in a friendlier UI, clearly showing individual test results as green or red: no more digging through logs. They also made failures much easier to debug, most notably, by displaying failure information, with links to logs, directly in the error message. For example, if Takeout failed to fetch a file from Gmail, the test would dynamically construct a link that searched for that file’s ID in the Takeout logs and include it in the test failure message. This automated much of the debugging process for product plug-in engineers and required less of the Takeout team’s assistance in sending them logs, as demonstrated in [Figure 23-3](#_bookmark2091).

**團隊所做的**。團隊將測試重構為一個動態的、基於配置的套件（使用一個引數化的測試執行器），在一個更友好的使用者介面中報告結果，清楚地顯示單個測試結果為綠色或紅色：不再翻閱日誌。他們還使失敗變得更容易除錯，最明顯的是，在錯誤資訊中直接顯示失敗資訊，並提供日誌連結。例如，如果Takeout從Gmail獲取檔案失敗，測試將動態地建構一個連結，在Takeout日誌中搜索該檔案的ID，並將其包含在測試失敗資訊中。如圖23-3所示，這使產品外掛工程師的大部分除錯過程自動化，並在向他們傳送日誌時不再需要Takeout團隊的協助。

![Figure 23-3](./images/Figure%2023-3.png)

*Figure* *23-3.* *The* *team’s* *involvement* *in* *debugging* *client* *failures*

**Lesson learned.** Accessible, actionable feedback from CI reduces test failures and improves productivity. These initiatives reduced the Takeout team’s involvement in debugging client (product plug-in) test failures by 35%.

**經驗教訓。**來自CI的可訪問、可操作的反饋減少了測試失敗，提高了生產力。這些舉措使Takeout團隊參與除錯客戶（產品外掛）測試失敗的情況減少了35%。

#### Scenario #3: Debugging “all of Google” 情景#3：除錯 "所有谷歌"

**Problem:** An interesting side effect of the Takeout CI that the team did not anticipate was that, because it verified the output of 90-some odd end-user–facing products, in the form of an archive, they were basically testing “all of Google” and catching issues that had nothing to do with Takeout. This was a good thing—Takeout was able to help contribute to the quality of Google’s products overall. However, this introduced a problem for their CI processes: they needed better failure isolation so that they could determine which problems were in their build (which were the minority) and which lay in loosely coupled microservices behind the product APIs they called.

**問題：**Takeout CI的一個有趣的副作用是團隊沒有預料到的，因為它以歸檔的形式驗證了90多個面向終端使用者的產品的輸出，他們基本上是在測試 "所有的Google產品"，捕捉與Takeout無關的問題。這是一件好事--Takeout能夠幫助提高谷歌產品的整體品質。然而，這給他們的CI流程帶來了一個問題：他們需要更好的故障隔離，以便他們能夠確定哪些問題是在他們的建構中（哪些是少數），哪些是在他們呼叫的產品API背後鬆散耦合的微服務中。

**What the team did.** The team’s solution was to run the exact same test suite continuously against production as it already did in its post-submit CI. This was cheap to implement and allowed the team to isolate which failures were new in its build and which were in production; for instance, the result of a microservice release somewhere else “in Google.”

**團隊所做的**。該團隊的解決方案是針對生產持續執行完全相同的測試套件，正如它在提交後CI中所做的那樣。這樣做的成本很低，並允許團隊隔離哪些故障是在其建構中出現的，哪些是在生產中出現的；例如，微服務發佈的結果“在谷歌的其他地方”。

**Lesson learned.** Running the same test suite against prod and a post-submit CI (with newly built binaries, but the same live backends) is a cheap way to isolate failures.

**經驗教訓**。對生產環境和提交後的CI執行相同的測試套件（使用新建構的二進位制檔案，但相同的實時後端）是隔離故障的廉價方法。

**Remaining challenge.** Going forward, the burden of testing “all of Google” (obviously, this is an exaggeration, as most product problems are caught by their respective teams) grows as Takeout integrates with more products and as those products become more complex. Manual comparisons between this CI and prod are an expensive use of the Build Cop’s time.

**仍然存在的挑戰。**展望未來，隨著Takeout與更多的產品整合，以及這些產品變得更加複雜，測試 "所有谷歌"（顯然，這是一個誇張的說法，因為大多數產品問題都是由他們各自的團隊發現的）的負擔越來越重。在這個CI和prod之間進行手動比較是對Build Cop時間的昂貴使用。

**Future improvement.** This presents an interesting opportunity to try hermetic testing with record/replay in Takeout’s post-submit CI. In theory, this would eliminate failures from backend product APIs surfacing in Takeout’s CI, which would make the suite more stable and effective at catching failures in the last two hours of Takeout changes—which is its intended purpose.

## Scenario #4: Keeping it green  場景4：保持綠色

**Problem:** As the platform supported more product plug-ins, which each included end-to-end tests, these tests would fail and the end-to-end test suites were nearly always broken. The failures could not all be immediately fixed. Many were due to bugs in product plug-in binaries, which the Takeout team had no control over. And some failures mattered more than others—low-priority bugs and bugs in the test code did not need to block a release, whereas higher-priority bugs did. The team could easily disable tests by commenting them out, but that would make the failures too easy to forget about.

**問題**：隨著平臺支援更多的產品外掛，每個外掛都包括端到端的測試，這些測試會失敗，端到端的測試套件幾乎總是被破壞。這些故障不可能都被立即修復。許多故障是由於產品外掛二進位制檔案中的錯誤，Takeout 團隊無法控制這些錯誤。有些故障比其他故障更重要--低優先順序的bug和測試程式碼中的bug不需要阻止發佈，而高優先順序的bug需要阻止。團隊可以很容易地透過註釋它們來禁用測試，但這將使失敗者很容易忘記。

One common source of failures: tests would break when product plug-ins were rolling out a feature. For example, a playlist-fetching feature for the YouTube plug-in might be enabled for testing in dev for a few months before being enabled in prod. The Takeout tests only knew about one result to check, so that often resulted in the test needing to be disabled in particular environments and manually curated as the feature rolled out.

一個常見的失敗原因是：當產品外掛推出一個功能時，測試會中斷。例如，YouTube外掛的播放列表獲取功能可能在開發階段啟用了幾個月的測試，然後才在生產階段啟用。Takeout測試只知道要檢查一個結果，所以這往往導致測試需要在特定的環境中被禁用，並在功能推出時被手動修復。

**What the team did.** The team came up with a strategic way to disable failing tests by tagging them with an associated bug and filing that off to the responsible team (usually a product plug-in team). When a failing test was tagged with a bug, the team’s testing framework would suppress its failure. This allowed the test suite to stay green and still provide confidence that everything else, besides the known issues, was passing, as illustrated in [Figure 23-4](#_bookmark2092).

**團隊所做的**。該團隊提出了一種禁用失敗測試的戰略方法，方法是使用相關錯誤標記失敗測試，並將其提交給負責的團隊（通常是產品外掛團隊）。當失敗的測試被標記為錯誤時，團隊的測試框架將抑制其失敗。這允許測試套件保持綠色，並且仍然提供信心，證明除已知問題外的所有其他問題都通過了，如圖23-4所示。

![Figure 23-4](./images/Figure%2023-4.png)

*Figure* *23-4.* *Achieving* *greenness* *through* *(responsible)* *test* *disablement* *透過（責任人）測試禁用來實現綠色*

For the rollout problem, the team added capability for plug-in engineers to specify the name of a feature flag, or ID of a code change, that enabled a particular feature along with the output to expect both with and without the feature. The tests were equipped to query the test environment to determine whether the given feature was enabled there and verified the expected output accordingly.

對於推廣問題，團隊增加了外掛工程師指定功能標誌名稱或程式碼更改ID的功能，該功能使特定功能和輸出能夠同時使用和不使用該功能。測試配備了查詢測試環境的功能，以確定給定的功能是否在那裡啟用，並相應地驗證預期輸出。

When bug tags from disabled tests began to accumulate and were not updated, the team automated their cleanup. The tests would now check whether a bug was closed by querying our bug system’s API. If a tagged-failing test actually passed and was passing for longer than a configured time limit, the test would prompt to clean up the tag (and mark the bug fixed, if it wasn’t already). There was one exception for this strategy: flaky tests. For these, the team would allow a test to be tagged as flaky, and the system wouldn’t prompt a tagged “flaky” failure for cleanup if it passed.

當被禁用的測試的bug標籤開始積累並且不被更新時，該團隊將其清理自動化。測試現在會透過查詢我們的錯誤系統的API來檢查一個錯誤是否被關閉。如果一個被標記為失敗的測試實際通過了，並且透過的時間超過了配置的時間限制，測試就會提示清理標籤（如果還沒有被修復的話，就標記為bug修復）。這個策略有一個例外：不穩定的測試。對於這些，團隊將允許測試被標記為不穩定，如果測試通過了，系統不會提示清理標記的 "不穩定 "故障。

These changes made a mostly self-maintaining test suite, as illustrated in [Figure 23-5](#_bookmark2093).‘

’這些變化使得測試套件大多是自我維護的，如圖23-5所示。

![Figure 23-5](./images/Figure%2023-5.png)

*Figure 23-5. Mean time to close bug, after fix submitted* *提交修復程式後關閉bug的平均時間*

**Lessons learned.** Disabling failing tests that can’t be immediately fixed is a practical approach to keeping your suite green, which gives confidence that you’re aware of all test failures. Also, automating the test suite’s maintenance, including rollout management and updating tracking bugs for fixed tests, keeps the suite clean and prevents technical debt. In DevOps parlance, we could call the metric in [Figure 23-5 ](#_bookmark2093)MTTCU: mean time to clean up.

**經驗教訓。**禁用無法立即修復的失敗測試是保持套件綠色的一種切實可行的方法，這使人相信你知道所有的測試失敗。另外，自動化測試套件的維護，包括推出管理和更新追蹤固定測試的bug，保持套件的清潔，防止技術債務。用DevOps的說法，我們可以把圖23-5MTTCU中的指標稱為：平均清理時間。

**Future improvement.** Automating the filing and tagging of bugs would be a helpful next step. This is still a manual and burdensome process. As mentioned earlier, some of our larger teams already do this.

**未來的改進。**自動歸檔和標記bug將是一個有用的下一步。這仍然是一個手動和繁重的過程。正如前面提到的，我們的一些大型團隊已經這樣做了。

**Further challenges.** The scenarios we’ve described are far from the only CI challenges faced by Takeout, and there are still more problems to solve. For example, we mentioned the difficulty of isolating failures from upstream services in [“CI Challenges” on](#_bookmark2059) [page 490](#_bookmark2059). This is a problem that Takeout still faces with rare breakages originating with upstream services, such as when a security update in the streaming infrastructure used by Takeout’s “Drive folder downloads” API broke archive decryption when it deployed to production. The upstream services are staged and tested themselves, but there is no simple way to automatically check with CI if they are compatible with Takeout after they’re launched into production. An initial solution involved creating an “upstream staging” CI environment to test production Takeout binaries against the staged versions of their upstream dependencies. However, this proved difficult to maintain, with additional compatibility issues between staging and production versions.

**進一步的挑戰。**我們所描述的場景遠不是Takeout所面臨的唯一的CI挑戰，還有更多問題需要解決。例如，我們在第490頁的 "CI挑戰 "中提到了從上游服務隔離故障的困難。這是Takeout仍然面臨的一個問題，即源於上游服務的罕見故障，例如Takeout的“驅動器資料夾下載”API使用的流式基礎結構中的安全更新在部署到生產環境時破壞了存檔解密。上游服務都是經過階段性測試的，但沒有簡單的方法在它們投入生產後用CI自動檢查它們是否與Takeout相容。最初的解決方案是建立一個 "上游臨時 "的CI環境，根據上游依賴的暫存版本測試Takeout的生產二進位制檔案。然而，這被證明是很難維護的，在臨時版本和生產版本之間存在著額外的相容性問題。

### But I Can’t Afford CI  但我用不起CI費用

You might be thinking that’s all well and good, but you have neither the time nor money to build any of this. We certainly acknowledge that Google might have more resources to implement CI than the typical startup does. Yet many of our products have grown so quickly that they didn’t have time to develop a CI system either (at least not an adequate one).

你可能會想，這一切都很好，但你既沒有時間也沒有錢來建立這些。我們當然承認，谷歌可能比一般的創業公司擁有更多的資源來實施CI。然而，我們的許多產品成長得如此之快，以至於他們也沒有時間去開發一個CI系統（至少不是一個合適的系統）。

In your own products and organizations, try and think of the cost you are already paying for problems discovered and dealt with in production. These negatively affect the end user or client, of course, but they also affect the team. Frequent production fire-fighting is stressful and demoralizing. Although building out CI systems is expensive, it’s not necessarily a new cost as much as a cost shifted left to an earlier— and more preferable—stage, reducing the incidence, and thus the cost, of problems occurring too far to the right. CI leads to a more stable product and happier developer culture in which engineers feel more confident that “the system” will catch problems, and they can focus more on features and less on fixing.

在你自己的產品和組織中，試著想想你已經為在生產中發現和處理的問題支付了多少成本。這些問題當然會對終端使用者或客戶產生負面影響，但它們也會影響到團隊。頻繁的生產救火是一種壓力和士氣的體現。儘管建立CI系統是昂貴的，但它不一定是一個新的成本，而是將成本轉移到一個更早的、更可取的階段，減少問題的發生率，從而減少成本，因為問題發生在右邊。CI帶來了更穩定的產品和更快樂的開發運營文化，在這種文化中，工程師更相信“系統”會發現問題，他們可以更多地關注功能，而不是修復問題。

## Conclusion 總結

Even though we’ve described our CI processes and some of how we’ve automated them, none of this is to say that we have developed perfect CI systems. After all, a CI system itself is just software and is never complete and should be adjusted to meet the evolving demands of the application and engineers it is meant to serve. We’ve tried to illustrate this with the evolution of Takeout’s CI and the future areas of improvement we point out.

儘管我們已經描述了我們的 CI 流程和一些自動化的方法，但這並不是說我們已經開發了完美的 CI 系統。畢竟，CI系統本身只是一個軟體，永遠不會完整，應該進行調整以滿足應用程式和工程師不斷變化的需求。我們試圖用Takeout的CI的演變和我們指出的未來改進領域來說明這一點。

## TL;DRs  內容提要

- A CI system decides what tests to use, and when.

- CI systems become progressively more necessary as your codebase ages and grows in scale.

- CI should optimize quicker, more reliable tests on presubmit and slower, less deterministic tests on post-submit.

- Accessible, actionable feedback allows a CI system to become more efficient.


- CI系統決定使用什麼測試以及何時使用。

- 隨著程式碼函式庫的老化和規模的擴大，CI系統變得越來越有必要。

- CI應該在提交前優化更快、更可靠的測試，在提交後優化更慢、更不確定的測試。

- 可訪問、可操作的反饋使CI系統變得更加有效。



