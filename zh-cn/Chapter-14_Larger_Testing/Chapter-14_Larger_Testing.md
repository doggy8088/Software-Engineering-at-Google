
**CHAPTER 14**

# Larger Testing

# 第十四章 大型測試

**Written by  Written by Joseph Graves**

**Edited by Lisa Carey**

In previous chapters, we have recounted how a testing culture was established at Google and how small unit tests became a fundamental part of the developer workflow. But what about other kinds of tests? It turns out that Google does indeed use many larger tests, and these comprise a significant part of the risk mitigation strategy necessary for healthy software engineering. But these tests present additional challenges to ensure that they are valuable assets and not resource sinks. In this chapter, we」ll discuss what we mean by 『larger tests,』 when we execute them, and best practices for keeping them effective.

在前幾章中，我們已經講述了測試文化是如何在Google建立的，以及小型單元測試是如何成為開發人員工作流程的基本組成部分。那麼其他型別的測試呢？事實證明，Google確實使用了許多大型測試，這些測試構成了健康的軟體工程所需的風險緩解策略的重要組成部分。但是想要確保它們是有價值的資產而不是資源黑洞，那麼這些測試面臨了更多的挑戰。在這一章中，我們將討論什麼是 "大型測試"，什麼時候執行這些測試，以及保持其有效性的最佳做法。

## What Are Larger Tests? 什麼是大型測試？

As mentioned previously, Google has specific notions of test size. Small tests are restricted to one thread, one process, one machine. Larger tests do not have the same restrictions. But Google also has notions of test scope. A unit test necessarily is of smaller scope than an integration test. And the largest-scoped tests (sometimes called end-to-end or system tests) typically involve several real dependencies and fewer test doubles.

如前所述，谷歌對測試規模有特定的概念。小型測試僅限於單線程、單程序、單伺服器。較大的測試沒有相同的限制。但谷歌同樣定義了測試的範圍。單元測試的範圍必然比整合測試的範圍小。而最大範圍的測試（有時被稱為端到端或系統測試）通常涉及多個實際依賴項和較少的測試替身。（`Test Double`是在Martin Fowler的文章[Test Double](https://martinfowler.com/bliki/TestDouble.html)中，Gerard Meszaros提出了這個概念。雖然是06年的文章了，但裡面的概念並不過時。這篇文章提到`Test Double`只是一個通用的詞，代表為了達到測試目的並且減少被測試物件的依賴，使用『替身』代替一個真實的依賴物件，從而保證了測試的速度和穩定性。統一翻譯為測試替代）

Larger tests are many things that small tests are not. They are not bound by the same constraints; thus, they can exhibit the following characteristics:

- They may be slow. Our large tests have a default timeout of 15 minutes or 1 hour, but we also have tests that run for multiple hours or even days.
- They may be nonhermetic. Large tests may share resources with other tests and traffic.
- They may be nondeterministic. If a large test is nonhermetic, it is almost impossible to guarantee determinism: other tests or user state may interfere with it.

較大的測試有許多是小型測試所不具備的內容。它們受的約束不同；因此，它們可以表現出以下特徵：

- 它們可能很慢。我們的大型測試的預設時長時間為15分鐘或1小時，但我們也有執行數小時甚至數天的測試。
- 它們可能是不具備封閉性。大型測試可能與其他測試和流量共享資源。
- 它們可能是不確定的。如果大型測試是非密封的，則幾乎不可能保證確定性：其他測試或使用者狀態可能會干擾它。

So why have larger tests? Reflect back on your coding process. How do you confirm that the programs you write actually work? You might be writing and running unit tests as you go, but do you find yourself running the actual binary and trying it out yourself? And when you share this code with others, how do they test it? By running your unit tests, or by trying it out themselves?

那麼，為什麼要進行大型測試？回想一下你的編碼過程。你是如何確認你寫的程式真的能工作的？你可能邊寫邊執行單元測試，但你是否發現自己在執行實際的二進位檔案並親自體驗？而當你與他人分享這些程式碼時，他們是如何測試的呢？是透過執行你的單元測試，還是透過自己體驗？

Also, how do you know that your code continues to work during upgrades? Suppose that you have a site that uses the Google Maps API and there」s a new API version. Your unit tests likely won」t help you to know whether there are any compatibility issues. You」d probably run it and try it out to see whether anything broke.

另外，你怎麼知道你的程式碼在升級時還能繼續工作？假設你有一個使用谷歌地圖API的網站，有一個新的API版本。你的單元測試很可能無法幫助你知道是否有任何相容性問題。你可能會執行它，試一試，看看是否有什麼故障。

Unit tests can give you confidence about individual functions, objects, and modules, but large tests provide more confidence that the overall system works as intended. And having actual automated tests scales in ways that manual testing does not.

單元測試可以讓你對單個功能、物件和模組有信心，但大型測試可以讓你對整個系統按預期工作更有信心。並且擁有實際的自動化測試能以手動測試無法比擬的方式擴充套件。

### Fidelity 模擬度

The primary reason larger tests exist is to address *fidelity*. Fidelity is the property by which a test is reflective of the real behavior of the system under test (SUT).

大型測試存在的主要是為了提高測試的模擬度。模擬度是測試反映被測系統（SUT）真實行為的屬性。

One way of envisioning fidelity is in terms of the environment. As Figure 14-1 illustrates, unit tests bundle a test and a small portion of code together as a runnable unit, which ensures the code is tested but is very different from how production code runs. Production itself is, naturally, the environment of highest fidelity in testing. There is also a spectrum of interim options. A key for larger tests is to find the proper fit, because increasing fidelity also comes with increasing costs and (in the case of production) increasing risk of failure.

一種設想模擬度的方法是在環境方面。如圖14-1所示，單元測試將測試和一小部分程式碼捆綁在一起作為一個可執行的單元，這確保了程式碼得到測試，但與生產程式碼的執行方式有很大不同。產品本身才是測試中模擬度最高的環境。也有一系列的臨時選項。大型測試的一個關鍵是要找到適當的契合點，因為提高模擬度也伴隨著成本的增加和（線上上的情況下）故障風險的增加。

![Figure 14-1](./images/Figure%2014-1.png)

*Figure 14-1. Scale of increasing fidelity* *圖14-1 環境模擬度遞增的尺度*

Tests can also be measured in terms of how faithful the test content is to reality. Many handcrafted, large tests are dismissed by engineers if the test data itself looks unrealistic. Test data copied from production is much more faithful to reality (having been captured that way), but a big challenge is how to create realistic test traffic *before* launching the new code. This is particularly a problem in artificial intelligence (AI), for which the 『seed』 data often suffers from intrinsic bias. And, because most data for unit tests is handcrafted, it covers a narrow range of cases and tends to conform to the biases of the author. The uncovered scenarios missed by the data represent a fidelity gap in the tests.

測試也可以用測試內容對現實的模擬度程度來衡量。如果測試資料本身看起來不真實，許多手工設定的大型測試就會被工程師摒棄。從生產中複製的測試資料模擬度更高（以這種方式捕獲），但一個很大的挑戰是如何在*啟動新程式碼之前*建立真實的測試流量。這在人工智慧（AI）中尤其是一個問題，因為 "種子 "資料經常受到內在偏見的影響。而且，由於大多數單元測試的資料是手工設定的，它涵蓋的案例範圍很窄，並傾向於符合作者的偏見。資料所遺漏的場景代表了測試中的模擬度差距。

### Common Gaps in Unit Tests 單元測試中常見的問題

Larger tests might also be necessary where smaller tests fail. The subsections that follow present some particular areas where unit tests do not provide good risk mitigation coverage.

如果較小的測試失敗，也可能需要進行較大的測試。下面的小節介紹了單元測試無法提供良好風險緩解覆蓋一些特定領域的範例。

#### Unfaithful doubles 模擬度不足的測試替代

A single unit test typically covers one class or module. Test doubles (as discussed in Chapter 13) are frequently used to eliminate heavyweight or hard-to-test dependencies. But when those dependencies are replaced, it becomes possible that the replacement and the doubled thing do not agree.

一個單元測試通常覆蓋一個類或模組。測試替代（如第13章所討論的）經常被用來消除重量級或難以測試的依賴項。但是當這些依賴關係被替換時，就有可能出現替換後的東西和被替換的東西不對應·的情況。

Almost all unit tests at Google are written by the same engineer who is writing the unit under test. When those unit tests need doubles and when the doubles used are mocks, it is the engineer writing the unit test defining the mock and its intended behavior. But that engineer usually did *not* write the thing being mocked and can be misinformed about its actual behavior. The relationship between the unit under test and a given peer is a behavioral contract, and if the engineer is mistaken about the actual behavior, the understanding of the contract is invalid.

在谷歌，幾乎所有的單元測試都是由編寫被測單元的工程師編寫的。當這些單元測試需要替代時，當使用的替代是模擬時，是編寫單元測試的工程師在定義模擬和它的預期行為。但該工程師通常*沒有*寫被模擬的東西，因此可能對其實際行為有誤解。被測單元與給定對等方之間的關係是一種行為契約，如果工程師對實際行為有誤解，則對契約的理解無效。

Moreover, mocks become stale. If this mock-based unit test is not visible to the author of the real implementation and the real implementation changes, there is no signal that the test (and the code being tested) should be updated to keep up with the changes.

此外，模擬會變得過時。如果實際實現的作者看不到這個基於模擬的單元測試，並且實際實現發生了變化，那麼就沒有訊號表明測試（及被測試的程式碼）需要更新以適應這些變化。

Note that, as mentioned in [Chapter 13], if teams provide fakes for their own services, this concern is mostly alleviated.

請注意，正如在第13章中提到的，如果團隊為他們自己的服務提供模擬，這種擔憂大多會得到緩解。

#### Configuration issues 設定問題

Unit tests cover code within a given binary. But that binary is typically not completely self-sufficient in terms of how it is executed. Usually a binary has some kind of deployment configuration or starter script. Additionally, real end-user-serving production instances have their own configuration files or configuration databases.

單元測試涵蓋了給定二進位中的程式碼。但該二進位檔案在如何執行方面通常不是完全自洽的。通常情況下，二進位檔案有某種部署設定或啟動指令碼。此外，真正為終端使用者服務的生產實例有它們自己的設定檔案或設定資料庫。

If there are issues with these files or the compatibility between the state defined by these stores and the binary in question, these can lead to major user issues. Unit tests alone cannot verify this compatibility.[^1] Incidentally, this is a good reason to ensure that your configuration is in version control as well as your code, because then, changes to configuration can be identified as the source of bugs as opposed to introducing random external flakiness and can be built in to large tests.

如果這些檔案存在問題，或者這些儲存定義的狀態與目標二進位檔案之間存在相容性問題，則可能會導致重大的使用者故障。單元測試不能驗證這種相容性。順便說一下，這是一個很好的理由，確保你的設定和你的程式碼一樣在版本控制中，因為這樣，設定的變更可以被識別為bug的來源，而不是引入隨機的外部碎片，並且可以在大型測試中建立。

At Google, configuration changes are the number one reason for our major outages. This is an area in which we have underperformed and has led to some of our most embarrassing bugs. For example, there was a global Google outage back in 2013 due to a bad network configuration push that was never tested. Configurations tend to be written in configuration languages, not production code languages. They also often have faster production rollout cycles than binaries, and they can be more difficult to test. All of these lead to a higher likelihood of failure. But at least in this case (and others), configuration was version controlled, and we could quickly identify the culprit and mitigate the issue.

在谷歌，設定變更是我們重大故障的頭號原因。這是一個我們表現不佳的領域，並導致了我們一些最尷尬的錯誤。例如，2013年，由於一次從未測試過的糟糕網路設定推送，谷歌出現了一次全球停機。設定通常使用設定語言編寫，具備更快的部署週期，但也更難測試。所有這些都會導致更高的失敗可能性。但至少在這種情況下（和其他情況下），設定訊息被納入了版本控制，我們可以快速識別故障並緩解問題。

> [^1]:	See 『Continuous Delivery』 on page 483 and Chapter 25 for more information.
>
> 1   有關更多訊息，請參見第483頁和第25章的『連續交付』。

#### Issues that arise under load 高負載導致的問題

At Google, unit tests are intended to be small and fast because they need to fit into our standard test execution infrastructure and also be run many times as part of a frictionless developer workflow. But performance, load, and stress testing often require sending large volumes of traffic to a given binary. These volumes become difficult to test in the model of a typical unit test. And our large volumes are big, often thousands or millions of queries per second (in the case of ads, [real-time bidding](https://oreil.ly/brV5-))!

在谷歌，單元測試的目的是小而快，因為它們需要適配標準測試執行基礎設施，也可以作為順暢的開發人員工作流程的一部分多次執行。但效能、負載和壓力測試往往需要向一個特定的二進位檔案傳送大量的流量。這些流量在典型的單元測試模型中變得難以製造。而我們的大流量是很大的，往往是每秒數千或數百萬次的查詢（在廣告的情況下，實時競價）!

#### Unanticipated behaviors, inputs, and side effects 非預期的行為、投入和副作用

Unit tests are limited by the imagination of the engineer writing them. That is, they can only test for anticipated behaviors and inputs. However, issues that users find with a product are mostly unanticipated (otherwise it would be unlikely that they would make it to end users as issues). This fact suggests that different test techniques are needed to test for unanticipated behaviors.

單元測試受限於編寫它們的工程師的想象能力。也就是說，他們只能測試預期的行為和輸入。然而，使用者在產品中發現的問題大多是未預料到的（否則，他們不太可能將其作為問題送出給最終使用者）。這一事實表明，需要不同的測試技術來測試非預期的行為。

[Hyrum」s Law](http://hyrumslaw.com/)is an important consideration here: even if we could test 100% for conformance to a strict, specified contract, the effective user contract applies to all visible behaviors, not just a stated contract. It is unlikely that unit tests alone test for all visible behaviors that are not specified in the public API.

海勒姆定律在這裡是一個重要的考慮因素：即使我們可以100%測試是否符合嚴格的規定契約，有效的使用者契約也適用於所有可見的行為，而不僅僅是規定的契約。單元測試不太可能單獨測試公共API中未指定的所有可視行為。

#### Emergent behaviors and the 『vacuum effect』 突發行為和 "真空效應"

Unit tests are limited to the scope that they cover (especially with the widespread use of test doubles), so if behavior changes in areas outside of this scope, it cannot be detected. And because unit tests are designed to be fast and reliable, they deliberately eliminate the chaos of real dependencies, network, and data. A unit test is like a problem in theoretical physics: ensconced in a vacuum, neatly hidden from the mess of the real world, which is great for speed and reliability but misses certain defect categories.

單元測試僅限於它們所覆蓋的範圍內（特別是隨著測試替代的廣泛使用），因此如果超過此範圍發生行為變化，則無法檢測到。由於單元測試被設計為快速可靠，它們設計成遮蔽外界的副作用及噪聲：比如真實依賴、網路和數據。單元測試就像理論物理中的一個問題：執行在真空中，巧妙地隱藏在現實世界的混亂中，這有助於提高速度和可靠性，但忽略了某些缺陷類別。

### Why Not Have Larger Tests? 為什麼不進行大型測試？

In earlier chapters, we discussed many of the properties of a developer-friendly test. In particular, it needs to be as follows:

- *Reliable*  
	It must not be flaky and it must provide a useful pass/fail signal.
- *Fast*  
	It needs to be fast enough to not interrupt the developer workflow.
- *Scalable*  
	Google needs to be able to run all such useful affected tests efficiently for presubmits and for post-submits.

在前面的章節中，我們討論了對開發者友好的測試的許多屬性。特別是，它需要做到以下幾點：
- *可靠的*  
	它不能是不確定的，它必須提供一個有用的透過/失敗訊號。
- *快速*  
	它需要足夠快，以避免中斷開發人員的工作流程。
- *可擴充性*  
	谷歌需要能夠有效地執行所有這些有用的受影響的測試，用於預送出和後送出。

Good unit tests exhibit all of these properties. Larger tests often violate all of these constraints. For example, larger tests are often flakier because they use more infrastructure than does a small unit test. They are also often much slower, both to set up as well as to run. And they have trouble scaling because of the resource and time requirements, but often also because they are not isolated—these tests can collide with one another.

好的單元測試展現出這些屬性。大型測試經常違反這些限制。例如，大型測試往往是脆弱的，因為它們比小單元測試使用更多的基礎設施。它們的設定和執行速度也往往慢得多。而且，由於資源和時間的要求，它們在擴充套件上有困難，但往往也因為它們不是孤立的——這些測試可能會相互衝突。

Additionally, larger tests present two other challenges. First, there is a challenge of ownership. A unit test is clearly owned by the engineer (and team) who owns the unit. A larger test spans multiple units and thus can span multiple owners. This presents a long-term ownership challenge: who is responsible for maintaining the test and who is responsible for diagnosing issues when the test breaks? Without clear ownership, a test rots.

此外，大型測試還帶來了另外兩個挑戰。首先，所有權是一個挑戰。單元測試顯然由擁有單元的工程師（和團隊）擁有。較大的測試跨越多個單元，因此可以跨越多個所有者。這帶來了一個長期的所有權挑戰：誰負責維護測試，誰負責在測試中斷時診斷問題？沒有明確的所有權，測試就會腐化。

The second challenge for larger tests is one of standardization (or the lack thereof). Unlike unit tests, larger tests suffer a lack of standardization in terms of the infrastructure and process by which they are written, run, and debugged. The approach to larger tests is a product of a system」s architectural decisions, thus introducing variance in the type of tests required. For example, the way we build and run A-B diff regression tests in Google Ads is completely different from the way such tests are built and run in Search backends, which is different again from Drive. They use different platforms, different languages, different infrastructures, different libraries, and competing testing frameworks.

大型測試的第二個挑戰是標準化問題（或缺乏標準化）。與單元測試不同，大型測試在編寫、執行和除錯的基礎設施和流程方面缺乏標準化。大型測試的方法是系統架構設計的產物，因此在所需的測試型別中引入了差異性。例如，我們在谷歌廣告中建立和執行A-B差異迴歸測試的方式與在搜尋後端建立和執行此類測試的方式完全不同，而搜尋後端又與雲儲存不同。他們使用不同的平台，不同的語言，不同的基礎設施，不同的函式庫，以及相互競爭的測試框架。

This lack of standardization has a significant impact. Because larger tests have so many ways of being run, they often are skipped during large-scale changes. (See Chapter 22.) The infrastructure does not have a standard way to run those tests, and asking the people executing LSCs to know the local particulars for testing on every team doesn」t scale. Because larger tests differ in implementation from team to team, tests that actually test the integration between those teams require unifying incompatible infrastructures. And because of this lack of standardization, we cannot teach a single approach to Nooglers (new Googlers) or even more experienced engineers, which both perpetuates the situation and also leads to a lack of understanding about the motivations of such tests.

這種缺乏標準化的情況有很大的影響。因為大型測試有許多執行方式，在大規模的變更中，它們經常被忽略。(見第22章) 基礎設施沒有一個標準的方式來執行這些測試，要求執行LSC的人員瞭解每個團隊測試的本地細節是不可行的。因為更大的測試在各個團隊的實施中是不同的，因此實際測試這些團隊之間整合的測試需要統一不相容的基礎架構。而且由於缺乏標準化，我們無法向Nooglers（新的Googlers）甚至更有經驗的工程師傳授統一的方法，這既使情況長期存在，也導致人們對這種測試的動機缺乏瞭解。

## Larger Tests at Google 谷歌的大型測試

When we discussed the history of testing at Google earlier (see Chapter 11), we mentioned how Google Web Server (GWS) mandated automated tests in 2003 and how this was a watershed moment. However, we actually had automated tests in use before this point, but a common practice was using automated large and enormous tests. For example, AdWords created an end-to-end test back in 2001 to validate product scenarios. Similarly, in 2002, Search wrote a similar 『regression test』 for its indexing code, and AdSense (which had not even publicly launched yet) created its variation on the AdWords test.

當我們在前面討論Google的測試歷史時（見第11章），我們討論了Google Web Server（GWS）如何在2003年強制執行自動化測試，以及這是一個分水嶺時刻。然而，在這之前，我們實際上已經有了自動化測試的使用，但一個普遍的做法是使用自動化的大型測試。例如，AdWords早在2001年就建立了一個端到端的測試來驗證產品方案。同樣，在2002年，搜尋部門為其索引程式碼寫了一個類似的 "迴歸測試"，而AdSense（當時甚至還沒有公開推出）在AdWords的測試上創造了它的變種。

Other 『larger』 testing patterns also existed circa 2002. The Google search frontend relied heavily on manual QA—manual versions of end-to-end test scenarios. And Gmail got its version of a 『local demo』 environment—a script to bring up an end-to- end Gmail environment locally with some generated test users and mail data for local manual testing.

其他 "較大"的測試模式也開始於2002年左右。谷歌搜尋前端在很大程度上依賴於手動質量檢查——端到端的測試場景的手動版本。Gmail得到了它的 "本地示範 "環境的版本——一個指令碼，在本地建立一個端到端的Gmail環境，其中有一些生成的測試用戶和郵件資料，用於本地手動測試。

When C/J Build (our first continuous build framework) launched, it did not distinguish between unit tests and other tests, but there were two critical developments that led to a split. First, Google focused on unit tests because we wanted to encourage the testing pyramid and to ensure the vast majority of written tests were unit tests. Second, when TAP replaced C/J Build as our formal continuous build system, it was only able to do so for tests that met TAP」s eligibility requirements: hermetic tests buildable at a single change that could run on our build/test cluster within a maximum time limit. Although most unit tests satisfied this requirement, larger tests mostly did not. However, this did not stop the need for other kinds of tests, and they have continued to fill the coverage gaps. C/J Build even stuck around for years specifically to handle these kinds of tests until newer systems replaced it.

當C/J Build（我們的第一個持續建立框架）推出時，它並沒有區分單元測試和其他測試，但有兩個關鍵的進展導致了區分兩者。首先，Google專注於單元測試，因為我們想鼓勵金字塔式測試，並確保絕大部分的測試是單元測試。第二，當TAP取代C/J Build成為我們正式的持續建立系統時，它只能為符合TAP資格要求的測試服務：可在一次修改中建立的密封測試，可在最大時間限制內執行在我們的建立/測試叢集上。儘管大多數單元測試滿足了這一要求，但大型測試大多不滿足。然而，這並沒有阻止對其他型別的測試的需求，而且它們一直在填補覆蓋率的空白。C/J Build甚至堅持了多年，專門處理這些型別的測試，直到更新的系統取代它。

### Larger Tests and Time 大型測試與時間

Throughout this book, we have looked at the influence of time on software engineering, because Google has built software running for more than 20 years. How are larger tests influenced by the time dimension? We know that certain activities make more sense the longer the expected lifespan of code, and testing of various forms is an activity that makes sense at all levels, but the test types that are appropriate change over the expected lifetime of code.

在本書中，我們一直在關注時間對軟體工程的影響，因為谷歌已經開發了執行20多年的軟體。大型測試是如何受到時間維度的影響的？我們知道，程式碼的生命週期越長，某些活行為就越有意義，各種形式的測試是一種在各個層面都有意義的活動，但適合的測試型別會隨著程式碼的生命週期而改變。

As we pointed out before, unit tests begin to make sense for software with an expected lifespan from hours on up. At the minutes level (for small scripts), manual testing is most common, and the SUT usually runs locally, but the local demo likely *is* production, especially for one-off scripts, demos, or experiments. At longer lifespans, manual testing continues to exist, but the SUTs usually diverge because the production instance is often cloud hosted instead of locally hosted.

正如我們之前所指出的，單元測試對於生命週期在幾小時以上的軟體開始有意義。在分鐘級別（小型指令碼），手動測試是最常見的，SUT通常在本地執行，但本地demo很可能*就是*產品，特別是對於一次性的指令碼、示範或實驗。在更長的生命期，手動測試繼續存在，但SUT通常是有差別的，因為生產實例通常是雲託管而不是本地託管。

The remaining larger tests all provide value for longer-lived software, but the main concern becomes the maintainability of such tests as time increases.

其餘大型測試都為生命週期較長的軟體提供了價值，但隨著時間的增加，主要的問題變成了這種測試的可維護性。

Incidentally, this time impact might be one reason for the development of the 『ice cream cone』 testing antipattern, as mentioned in the Chapter 11 and shown again in Figure 14-2.

順便說一句，這一時間衝擊可能是開發『冰淇淋筒』測試反模式的原因之一，如第11章所述，圖14-2再次顯示

![Figure 14-2](./images/Figure%2014-2.png)

*Figure 14-2. The ice cream cone testing antipattern*

When development starts with manual testing (when engineers think that code is meant to last only for minutes), those manual tests accumulate and dominate the initial overall testing portfolio. For example, it」s pretty typical to hack on a script or an app and test it out by running it, and then to continue to add features to it but continue to test it out by running it manually. This prototype eventually becomes functional and is shared with others, but no automated tests actually exist for it.

當開發從手動測試開始時（當工程師認為程式碼只能持續幾分鐘時），那些手動測試就會積累起來並主導最初的整體測試組合。例如，通常我們會修改指令碼或應用程式並透過執行來驗證修改有效，然後繼續向其新增功能並透過手動執行來測試它，這是非常典型的。長此以往，原型產品的功能會越來越完整，直至可與其他人共享，但實際上不存在針對它的自動測試。

Even worse, if the code is difficult to unit test (because of the way it was implemented in the first place), the only automated tests that can be written are end-to-end ones, and we have inadvertently created 『legacy code』 within days.

更糟糕的是，如果程式碼很難進行單元測試（因為它最初的實現方式），那麼唯一可以編寫的自動化測試就是端到端的測試，並且我們在幾天內無意中建立了『遺留程式碼』。

It is *critical* for longer-term health to move toward the test pyramid within the first few days of development by building out unit tests, and then to top it off after that point by introducing automated integration tests and moving away from manual end- to-end tests. We succeeded by making unit tests a requirement for submission, but covering the gap between unit tests and manual tests is necessary for long-term health.

在開發的頭幾天，透過建立單元測試，向金字塔式測試邁進，然後在這之後透過引入自動化整合測試，減少手動執行的端到端測試，這對長期的穩定是*至關重要*的。我們成功地使單元測試成為送出的要求，但彌補單元測試和手工測試之間的差距對長期穩健是必要的。

#### Larger Tests at Google Scale 谷歌規模的大型測試

It would seem that larger tests should be more necessary and more appropriate at larger scales of software, but even though this is so, the complexity of authoring, running, maintaining, and debugging these tests increases with the growth in scale, even more so than with unit tests.

在軟體規模較大的情況下，大型測試似乎更有必要，也更合適，但即使如此，編寫、執行、維護和除錯這些測試的複雜性也會隨著規模的增長而增加，複雜度遠超過單元測試。

In a system composed of microservices or separate servers, the pattern of interconnections looks like a graph: let the number of nodes in that graph be our *N*. Every time a new node is added to this graph, there is a multiplicative effect on the number of distinct execution paths through it.

在由微服務或獨立伺服器組成的系統中，它們的互連模式形似一個圖，圖中的節點數我們用N來表示。每次向該圖新增新節點時，都會對透過該圖的不同執行路徑的數量產生乘法效應的倍增。

[Figure 14-3 ](#_bookmark1226)depicts an imagined SUT: this system consists of a social network with users, a social graph, a stream of posts, and some ads mixed in. The ads are created by advertisers and served in the context of the social stream. This SUT alone consists of two groups of users, two UIs, three databases, an indexing pipeline, and six servers. There are 14 edges enumerated in the graph. Testing all of the end-to-end possibilities is already difficult. Imagine if we add more services, pipelines, and databases to this mix: photos and images, machine learning photo analysis, and so on?

圖14-3描繪了一個想象中的SUT：這個系統由一個有用戶的社交網路、一個社交圖、一個feed流和一些混合廣告組成。廣告由廣告商建立，並在社交流的背景下提供服務。這個SUT單獨由兩組使用者、兩個UI、三個資料庫、一個索引通道和六個伺服器組成。圖中列舉了14條邊。測試所有端到端的可能性已經很困難了。想象一下，如果我們在這個組合中新增更多的服務、通道和數據函式庫：照片和影象、機器學習照片分析等等？

![Figure 14-3](./images/Figure%2014-3.png)

*Figure 14-3. Example of a fairly small SUT: a social network with advertising*

The rate of distinct scenarios to test in an end-to-end way can grow exponentially or combinatorially depending on the structure of the system under test, and that growth does not scale. Therefore, as the system grows, we must find alternative larger testing strategies to keep things manageable.

以端到端的方式測試的不同場景的速率可以指數增長或組合增長，這取決於被測系統的結構，並且這種增長不可持續。因此，隨著系統的發展，我們必須找到其他大型測試的測試策略，以保持測試的可管理性。

However, the value of such tests also increases because of the decisions that were necessary to achieve this scale. This is an impact of fidelity: as we move toward larger-*N* layers of software, if the service doubles are lower fidelity (1-epsilon), the chance of bugs when putting it all together is exponential in *N*. Looking at this example SUT again, if we replace the user server and ad server with doubles and those doubles are low fidelity (e.g., 10% accurate), the likelihood of a bug is 99% (1 – (0.1 ∗ 0.1)). And that」s just with two low-fidelity doubles.

然而，由於實現這一規模所需的決策，此類測試的價值也增加了。這是模擬度的一個影響：隨著我們向更大的N層軟體發展，如果服務的模擬度加倍（1ε），那麼當把所有的服務放在一起時，出現錯誤的機率是N的指數。再看看這個例子SUT，如果我們用測試替代來取代使用者伺服器和廣告伺服器，並且這些測試替代的模擬度較低（例如，10%的不準確度），出現錯誤的可能性為99%（1–（0.1 ∗ 0.1)). 這只是兩個低模擬度的替代。

Therefore, it becomes critical to implement larger tests in ways that work well at this scale but maintain reasonably high fidelity.

因此，以在這種規模下工作良好但保持合理高模擬度的方式實現更大的測試變得至關重要。

------

Tip:"The Smallest Possible Test" 提示："儘可能小的測試"
Even for integration tests,smaller is better-a handful of large tests is preferable to anenormous one.And,because the scope of a test is often coupled to the scope of theSUT,finding ways to make the SUT smaller help make the test smaller.

即便是整合測試，也是越小越好——幾個大型測試也比一個超大測試要好。而且，因為測試的範圍經常與SUT的範圍相聯系，找到使SUT變小的方法有助於使測試變小。

One way to achieve this test ratio when presented with a user journey that can requirecontributions from many internal systems is to "chain"tests,as illustrated inFigure 14-4,not specifically in their execution,but to create multiple smaller pairwiseintegration tests that represent the overall scenario.This is done by ensuring that theoutput of one test is used as the input to another test by persisting this output to adata repository.

當出現一個需要許多內部系統服務的使用者請求時，實現這種測試比率的一種方法是 "串聯"測試，如圖14-4所示，不是具體執行，而是建立多個較小的成對整合測試，代表整個場景。

![Figure 14-4](./images/Figure%2014-4.png)

Figure 14-4. Chained tests

## Structure of a Large Test 大型測試組成

Although large tests are not bound by small test constraints and could conceivably consist of anything, most large tests exhibit common patterns. Large tests usually consist of a workflow with the following phases:

- Obtain a system under test
- Seed necessary test data
- Perform actions using the system under test
- Verify behaviors

儘管大型測試不受小型測試約束的約束，並且可以由任何內容組成，但大多數大型測試都顯示出共同的模式。大型測試通常由具有以下階段的流程組成：

- 獲得被測試的系統 
- 必要的測試資料
- 使用被測系統執行操作
- 驗證行為

### The System Under Test 被測試的系統

One key component of large tests is the aforementioned SUT (see Figure 14-5). A typical unit test focuses its attention on one class or module. Moreover, the test code runs in the same process (or Java Virtual Machine [JVM], in the Java case) as the code being tested. For larger tests, the SUT is often very different; one or more separate processes with test code often (but not always) in its own process.

大型測試的一個關鍵組成部分是前述的SUT（見圖14-5）。一個典型的單元測試將關注點集中在一個類或模組上。此外，測試程式碼執行在與被測試程式碼相同的程序（在Java中，也可能在相同的Java虛擬機[JVM]中）。對於大型測試，SUT通常是非常不同的；一個或多個獨立的程序，測試程式碼通常（但不總是）在自己的程序中。

![Figure 14-5](./images/Figure%2014-5.png)

*Figure 14-5. An example system under test (SUT)*

At Google, we use many different forms of SUTs, and the scope of the SUT is one of the primary drivers of the scope of the large test itself (the larger the SUT, the larger the test). Each SUT form can be judged based on two primary factors:

- *Hermeticity*  
	This is the SUT」s isolation from usages and interactions from other components than the test in question. An SUT with high hermeticity will have the least exposure to sources of concurrency and infrastructure flakiness.
- *Fidelity*  
	The SUT」s accuracy in reflecting the production system being tested. An SUT with high fidelity will consist of binaries that resemble the production versions (rely on similar configurations, use similar infrastructures, and have a similar overall topology).

在谷歌，我們使用許多不同形式的SUT，而SUT的範圍是大型測試本身範圍的主要驅動因素之一（SUT越大，測試越大）。每種SUT形式都可以根據兩個主要因素來判斷。
- *封閉性*  
	這是SUT與相關測試之外的其他元件的使用和互動的隔離。具有高隔離性的SUT將具有最少的併發性和基礎架構脆弱性來源。
- *模擬度*  
	SUT反映被測生產系統的準確性。具有高模擬度的SUT將由與生產版本相似的二進位檔案組成（依賴於類似的設定，使用類似的基礎設施，並且具有類似的總體拓撲）。

Often these two factors are in direct conflict. Following are some examples of SUTs:

- *Single-process SUT*  
	The entire system under test is packaged into a single binary (even if in production these are multiple separate binaries). Additionally, the test code can be packaged into the same binary as the SUT. Such a test-SUT combination can be a 『small』 test if everything is single-threaded, but it is the least faithful to the production topology and configuration.
- *Single-machine SUT*  
	The system under test consists of one or more separate binaries (same as production) and the test is its own binary. But everything runs on one machine. This is used for 『medium』 tests. Ideally, we use the production launch configuration of each binary when running those binaries locally for increased fidelity.
- *Multimachine SUT*  
	The system under test is distributed across multiple machines (much like a production cloud deployment). This is even higher fidelity than the single-machine SUT, but its use makes tests 『large』 size and the combination is susceptible to increased network and machine flakiness.
- *Shared environments (staging and production)*  
	Instead of running a standalone SUT, the test just uses a shared environment. This has the lowest cost because these shared environments usually already exist, but the test might conflict with other simultaneous uses and one must wait for the code to be pushed to those environments. Production also increases the risk of end-user impact.
- *Hybrids*  
	Some SUTs represent a mix: it might be possible to run some of the SUT but have it interact with a shared environment. Usually the thing being tested is explicitly run but its backends are shared. For a company as expansive as Google, it is practically impossible to run multiple copies of all of Google」s interconnected services, so some hybridization is required.

這兩個因素常常是直接相互衝突的。以下是一些SUT的例子：

- *單程序SUT*  
	整個被測系統被打包成一個二進位檔案（即使在生產中這些是多個獨立的二進位檔案）。此外，測試程式碼可以被打包成與SUT相同的二進位檔案。如果所有測試都是單線程的，那麼這種測試SUT組合可能是一個『小』測試，但它對生產拓撲和設定模擬度最低。
- *單機SUT*  
	被測系統由一個或多個獨立的二進位檔案組成（與生產相同），測試有其獨立的二進位檔案。但一切都在一台機器上執行。這用於 "中等 "測試。理想情況下，在本地執行這些二進位檔案時，我們使用每個二進位檔案的生產啟動設定，以提高模擬度。
- *多機SUT*  
	被測系統分佈在多台機器上（很像生產雲部署）。這比單機SUT的模擬度還要高，但它的使用使得測試的規模 "很大"，而且這種組合很容易受到網路和機器脆弱程度的影響。
- *共享環境（預發和生產）*  
	測試只使用共享環境，而不是執行獨立的SUT。這具有最低的成本，因為這些共享環境通常已經存在，但是測試可能會與其他同時使用衝突，並且必須等待程式碼被推送到這些環境中。生產也增加了最終使用者受到影響的風險。
- *混合模式*  
	一些SUT代表了一種混合：可以執行一些SUT，但可以讓它與共享環境互動。通常被測試的東西是顯式執行的，但是它的後端是共享的。對於像谷歌這樣擴張的公司來說，實際上不可能執行所有谷歌互聯服務的多個副本，因此需要一些混合。

#### The benefits of hermetic SUTs 封閉式SUT的好處

The SUT in a large test can be a major source of both unreliability and long turnaround time. For example, an in-production test uses the actual production system deployment. As mentioned earlier, this is popular because there is no extra overhead cost for the environment, but production tests cannot be run until the code reaches that environment, which means those tests cannot themselves block the release of the code to that environment—the SUT is too late, essentially.

大型測試中的SUT可能是不可靠性和長執行時間的主要原因。例如，生產中的測試使用實際的生產系統部署。如前所述，這很流行，因為沒有額外的環境開銷成本，但在程式碼到達生產環境之前，生產測試無法執行，這意味著這些測試本身無法阻止將程式碼發布到生產環境--SUT本質上太晚了。

The most common first alternative is to create a giant shared staging environment and to run tests there. This is usually done as part of some release promotion process, but it again limits test execution to only when the code is available. As an alternative, some teams will allow engineers to 『reserve』 time in the staging environment and to use that time window to deploy pending code and to run tests, but this does not scale with a growing number of engineers or a growing number of services, because the environment, its number of users, and the likelihood of user conflicts all quickly grow.

最常見的第一種選擇是建立一個巨大的共享預發環境並在那裡執行測試。這通常是作為某些發布升級過程的一部分來完成的，但它再次將測試執行限制為僅當代碼可用時。作為一個替代方案，一些團隊允許工程師在預發環境中"保留 "時間，並使用該時間視窗來部署待定的程式碼和執行測試，但這並不能隨著工程師數量的增加或服務數量的增加而擴充套件，因為環境、使用者數量和使用者衝突的可能性都會迅速增加。

The next step is to support cloud-isolated or machine-hermetic SUTs. Such an environment improves the situation by avoiding the conflicts and reservation requirements for code release.

下一步是支援雲隔離的或機器密閉的SUT。這樣的環境透過避免程式碼發布的衝突和保留要求來改善情況。

------

*Case Study:Risks of testing in production and Webdriver Torso*  
*案例研究：生產中的測試風險和Webdriver Torso*

We mentioned that testing in production can be risky.One humorous episode resulting from testing in production was known as the Webdriver Torso incident.Weneeded a way to verify that video rendering in You Tube production was workingproperly and so created automated scripts to generate test videos,upload them,andverify the quality of the upload.This was done in a Google-owned YouTube channelcalled Webdriver Torso.But this channel was public,as were most of the videos.

我們提到，在生產中進行測試是有風險的。我們需要一種方法來驗證YouTube生產中的影片渲染是否正常，因此建立了自動指令碼來生成測試影片，上傳它們，並驗證上傳質量，這是在谷歌擁有的名為Webdriver Torso的YouTube中進行的。

Subsequently,this channel was publicized in an article at Wired,which led to itsspread throughout the media and subsequent efforts to solve the mystery.Finally,ablogger tied everything back to Google.Eventually,we came clean by having a bit offun with it,including a Rickroll and an Easter Egg,so everything worked out well.Butwe do need to think about the possibility of end-user discovery of any test data weinclude in production and be prepared for it.

後來，這個渠道在《連線》雜誌的一篇文章中被公佈，這導致它在媒體上傳播，隨後人們努力解開這個謎團。最後，我們透過與它進行一些互動，包括一個Rickroll和一個復活節彩蛋，所以一切都很順利。但我們確實需要考慮最終使用者發現我們在生產中包含的任何測試資料的可能性並做好準備。

----------

#### Reducing the size of your SUT at problem boundaries 減少問題邊界內SUT的範圍

There are particularly painful testing boundaries that might be worth avoiding. Tests that involve both frontends and backends become painful because user interface (UI) tests are notoriously unreliable and costly:
- UIs often change in look-and-feel ways that make UI tests brittle but do not actually impact the underlying behavior.
- UIs often have asynchronous behaviors that are difficult to test.

有一些特別痛苦的測試界限，值得避免。同時涉及前台和後台的測試變得很痛苦，因為使用者介面（UI）測試是出了名的不可靠和高成本：
- UI的外觀和感覺方式經常發生變化，使UI測試變得脆弱，但實際上不會影響底層行為。
- UI通常具有難以測試的非同步行為。

Although it is useful to have end-to-end tests of a UI of a service all the way to its backend, these tests have a multiplicative maintenance cost for both the UI and the backends. Instead, if the backend provides a public API, it is often easier to split the tests into connected tests at the UI/API boundary and to use the public API to drive the end-to-end tests. This is true whether the UI is a browser, command-line interface (CLI), desktop app, or mobile app.

儘管對服務的UI進行端到端測試非常有用，但這些測試會增加UI和後端的維護成本。相反，如果後端提供公共API，則通常更容易在UI/API邊界將測試拆分為連線的測試，並使用公共API驅動端到端測試。無論UI是瀏覽器、指令行介面（CLI）、桌面應用程式還是行動應用程式，都是如此。

Another special boundary is for third-party dependencies. Third-party systems might not have a public shared environment for testing, and in some cases, there is a cost with sending traffic to a third party. Therefore, it is not recommended to have automated tests use a real third-party API, and that dependency is an important seam at which to split tests.

另一個特殊的邊界是第三方依賴關係。第三方系統可能沒有用於測試的公共共享環境，在某些情況下，向第三方傳送流量會產生成本。因此，不建議讓自動對應的測試使用真正的第三方API，並且依賴性是分割測試的一個重要接點。

To address this issue of size, we have made this SUT smaller by replacing its databases with in-memory databases and removing one of the servers outside the scope of the SUT that we actually care about, as shown in Figure 14-6. This SUT is more likely to fit on a single machine.

為了解決規模問題，我們透過用記憶體資料庫替換它的資料庫，並移除SUT範圍之外的一個我們真正關心的伺服器，使這個SUT變得更小，如圖14-6所示。這個SUT更可能適合在一台機器上使用。

![Figure 14-6](./images/Figure%2014-6.png)

*Figure 14-6. A reduced-size SUT*

The key is to identify trade-offs between fidelity and cost/reliability, and to identify reasonable boundaries. If we can run a handful of binaries and a test and pack it all into the same machines that do our regular compiles, links, and unit test executions, we have the easiest and most stable 『integration』 tests for our engineers.

關鍵是要確定模擬度和成本/可靠性之間的權衡，並確定合理的邊界。如果我們能夠執行少量的二進位檔案和一個測試，並將其全部打包到進行常規編譯、連結和單元測試執行的同一台機器上，我們就能為我們的工程師提供最簡單、最穩定的 "整合 "測試。

#### Record/replay proxies 錄製/重放代理

In the previous chapter, we discussed test doubles and approaches that can be used to decouple the class under test from its difficult-to-test dependencies. We can also double entire servers and processes by using a mock, stub, or fake server or process with the equivalent API. However, there is no guarantee that the test double used actually conforms to the contract of the real thing that it is replacing.

在前一章中，我們討論了測試加倍和可用於將被測類與其難以測試的依賴項解耦的方法。我們還可以透過使用具有等效API的模擬、打樁或偽伺服器或程序來復制整個伺服器和程序。然而，無法保證所使用的測試替代實際上符合其所替換的真實物件的契約。

One way of dealing with an SUT」s dependent but subsidiary services is to use a test double, but how does one know that the double reflects the dependency」s actual behavior? A growing approach outside of Google is to use a framework for [consumer-driven contract](https://oreil.ly/RADVJ)tests. These are tests that define a contract for both the client and the provider of the service, and this contract can drive automated tests. That is, a client defines a mock of the service saying that, for these input arguments, I get a particular output. Then, the real service uses this input/output pair in a real test to ensure that it produces that output given those inputs. Two public tools for consumer-driven contract testing are [Pact Contract Testing](https://docs.pact.io/)and [Spring Cloud Contracts](https://oreil.ly/szQ4j). Google」s heavy dependency on protocol buffers means that we don」t use these internally.

處理SUT的依賴關係和附屬服務的一種方法是使用測試替代，但如何知道替代反映了依賴的實際行為？在谷歌之外，一種正在發展的方法是使用一個框架進行消費者驅動的合同測試。這些測試為客戶和服務的提供者定義了一個契約，這個契約可以驅動自動測試。也就是說，一個客戶定義了一個服務的模擬，說對於這些輸入引數，我得到一個特定的輸出。然後，真正的服務在真正的測試中使用這個輸入/輸出對，以確保它在這些輸入的情況下產生那個輸出。消費者驅動的合同測試的兩個公共工具是[Pact Contract Testing](https://docs.pact.io/)和[Spring Cloud Contracts](https://oreil.ly/szQ4j)。谷歌對protocol buffers的嚴重依賴意味著我們內部不使用這些工具。

At Google, we do something a little bit different. [Our most popular approach](https://oreil.ly/-wvYi)(for which there is a public API) is to use a larger test to generate a smaller one by recording the traffic to those external services when running the larger test and replaying it when running smaller tests. The larger, or 『Record Mode』 test runs continuously on post-submit, but its primary purpose is to generate these traffic logs (it must pass, however, for the logs to be generated). The smaller, or 『Replay Mode』 test is used during development and presubmit testing.

在谷歌，我們做的有些不同。我們最流行的方法（有公共API）是使用較大的測試生成較小的測試，方法是在執行較大的測試時錄製到這些外部服務的流量，並在執行較小的測試時重放流量。大型或『記錄模式』測試在送出後持續執行，但其主要目的是生成這些流量日誌（但必須透過才能生成日誌）。在開發和送出前測試過程中，使用較小的或『重放模式』測試。

One of the interesting aspects of how record/replay works is that, because of nondeterminism, requests must be matched via a matcher to determine which response to replay. This makes them very similar to stubs and mocks in that argument matching is used to determine the resulting behavior.

錄製/重放工作原理的一個有趣方面是，由於不確定性，必須透過對應器對應請求，以確定重放的回應。這使得它們與打樁和模擬非常相似，因為引數對應用於確定結果行為。

What happens for new tests or tests where the client behavior changes significantly? In these cases, a request might no longer match what is in the recorded traffic file, so the test cannot pass in Replay mode. In that circumstance, the engineer must run the test in Record mode to generate new traffic, so it is important to make running Record tests easy, fast, and stable.

新測試或用戶端行為發生顯著變化的測試會發生什麼情況？在這些情況下，請求可能不再與錄製的流量檔案中的內容對應，因此測試無法在重放模式下透過。在這種情況下，工程師必須以記錄模式執行測試以生成新的通訊量，因此使執行錄製測試變得簡單、快速和穩定非常重要。

### Test Data 測試資料

A test needs data, and a large test needs two different kinds of data:

- *Seeded data*  
	Data preinitialized into the system under test reflecting the state of the SUT at the inception of the test
- *Test traffic*  
	Data sent to the system under test by the test itself during its execution

測試需要資料，大型測試需要兩種不同的資料：

- *種子資料*  
	預先初始化到被測系統中的資料，反映測試開始時SUT的狀態
- *測試流量*  
	在測試執行過程中，由測試本身傳送至被測系統的資料。

Because of the notion of the separate and larger SUT, the work to seed the SUT state is often orders of magnitude more complex than the setup work done in a unit test. For example:

- *Domain data*  
	Some databases contain data prepopulated into tables and used as configuration for the environment. Actual service binaries using such a database may fail on startup if domain data is not provided.
- *Realistic baseline*  
	For an SUT to be perceived as realistic, it might require a realistic set of base data at startup, both in terms of quality and quantity. For example, large tests of a social network likely need a realistic social graph as the base state for tests: enough test users with realistic profiles as well as enough interconnections between those users must exist for the testing to be accepted.
- *Seeding APIs*  
	The APIs by which data is seeded may be complex. It might be possible to directly write to a datastore, but doing so might bypass triggers and checks performed by the actual binaries that perform the writes.

由於獨立的和更大的SUT的概念，SUT狀態的種子工作往往比單元測試中的設定工作要複雜得多。比如說：
- *領域資料*  
	一些資料庫包含預先填充到表中的資料，並作為環境的設定使用。如果不提供領域資料，使用這種資料庫的實際服務二進位檔案可能在啟動時失敗。
- *現實的基線*  
	要使SUT被認為是現實的，它可能需要在啟動時提供一組現實的基礎資料，包括質量和數量。例如，社交網路的大型測試可能需要一個真實的社交圖作為測試的基本狀態：必須有足夠多的具有真實設定檔案的測試用戶以及這些使用者之間的足夠互聯，才能接受測試。
- *種子APIs*  
	用於資料初始化的API可能相當複雜。也許可以直接寫入資料儲存，但這樣做可能會繞過由執行寫入的實際二進位檔案執行的觸發器和檢查。

Data can be generated in different ways, such as the following:

- *Handcrafted data*  
	Like for smaller tests, we can create test data for larger tests by hand. But it might require more work to set up data for multiple services in a large SUT, and we might need to create a lot of data for larger tests.
- *Copied data*  
	We can copy data, typically from production. For example, we might test a map of Earth by starting with a copy of our production map data to provide a baseline and then test our changes to it.
- *Sampled data*  
	Copying data can provide too much data to reasonably work with. Sampling data can reduce the volume, thus reducing test time and making it easier to reason about. 『Smart sampling』 consists of techniques to copy the minimum data necessary to achieve maximum coverage.

資料可以透過不同的方式產生，比如說以下幾種：

- *手工製作資料*  
	與小型測試一樣，我們可以手動建立大型測試的測試資料。但是在一個大型SUT中為多個服務設定資料可能需要更多的工作，並且我們可能需要為大型測試建立大量資料。
- *複製的資料*  
	我們可以複製資料，通常來自生產。例如，我們可以透過從生產地圖資料的副本開始測試地球地圖，以提供基線，然後測試我們對它的更改。
- *抽樣資料*  
	複製資料可能產生過多，難以有效處理的資料。取樣資料可以減少數量，從而減少測試時間，使其更容易推理。"智慧抽樣 "包括複製最小的資料以達到最大覆蓋率的技術。

### Verification 驗證

After an SUT is running and traffic is sent to it, we must still verify the behavior. There are a few different ways to do this:

- *Manual*  
	Much like when you try out your binary locally, manual verification uses humans to interact with an SUT to determine whether it functions correctly. This verification can consist of testing for regressions by performing actions as defined on a consistent test plan or it can be exploratory, working a way through different interaction paths to identify possible new failures.
	Note that manual regression testing does not scale sublinearly: the larger a system grows and the more journeys through it there are, the more human time is needed to manually test.
- *Assertions*  
	Much like with unit tests, these are explicit checks about the intended behavior of the system. For example, for an integration test of Google search of xyzzy, an assertion might be as follows:

```java 
assertThat(response.Contains("Colossal Cave"))
```

- *A/B comparison (differential)*  
	Instead of defining explicit assertions, A/B testing involves running two copies of the SUT, sending the same data, and comparing the output. The intended behavior is not explicitly defined: a human must manually go through the differences to ensure any changes are intended.

在SUT執行並向其傳送流量後，我們仍然必須驗證其行為。有幾種不同的方法可以做到這一點:

- *手動*  
	就像你在本地嘗試你的二進位檔案一樣，手動驗證使用人工與SUT互動以確定它的功能是否正確。這種驗證可以包括透過執行一致的測試計劃中定義的操作來測試迴歸，也可以是探索性的，透過不同的互動路徑來識別可能的新故障。
	需要注意的是，人工迴歸測試的規模化不是線性的：系統越大，透過它的操作越多，需要的人力測試時間就越多就越更多。
- *斷言*  
	與單元測試一樣，這些是對系統預期行為的明確檢查。例如，對於谷歌搜尋xyzzy的整合測試，一個斷言可能如下：

```
assertThat(response.Contains("Colossal Cave"))
```

- *A/B測試（差異）*  
	A/B測試不是定義顯式斷言，而是執行SUT的兩個副本，傳送相同的資料，並比較結果。未明確定義預期行為：人工必須手動檢查差異，以確保任何預期更改。

## Types of Larger Tests 大型測試的型別

We can now combine these different approaches to the SUT, data, and assertions to create different kinds of large tests. Each test then has different properties as to which risks it mitigates; how much toil is required to write, maintain, and debug it; and how much it costs in terms of resources to run.

我們現在可以將這些不同的方法組合到SUT、資料和斷言中，以建立不同型別的大型測試。然後，每項測試都有不同的屬性，可以降低哪些風險；編寫、維護和除錯它需要多少工作了；以及它在執行資源方面的成本。

What follows is a list of different kinds of large tests that we use at Google, how they are composed, what purpose they serve, and what their limitations are:

- Functional testing of one or more binaries
- Browser and device testing
- Performance, load, and stress testing
- Deployment configuration testing
- Exploratory testing
- A/B diff (regression) testing
- User acceptance testing (UAT)
- Probers and canary analysis
- Disaster recovery and chaos engineering
- User evaluation

下面是我們在谷歌使用的各種大型測試的清單，它們是如何組成的，它們的用途是什麼，它們的侷限性是什麼：

- 一個或多個二進位檔案的功能測試
- 瀏覽器和裝置測試
- 效能、負載和壓力測試
- 部署設定測試
- 探索性測試
- A/B對比（迴歸）測試
- 使用者驗收測試（UAT）
- 探針和金絲雀分析
- 故障恢復和混沌工程
- 使用者評價

Given such a wide number of combinations and thus a wide range of tests, how do we manage what to do and when? Part of designing software is drafting the test plan, and a key part of the test plan is a strategic outline of what types of testing are needed and how much of each. This test strategy identifies the primary risk vectors and the necessary testing approaches to mitigate those risk vectors.

考慮到如此廣泛的組合和如此廣泛的測試，我們如何管理做什麼以及何時做？軟體設計的一部分是制定測試計劃，測試計劃的關鍵部分是確定所需測試型別的戰略性概述及其數量。該測試策略確定了主要風險向量和緩解這些風險向量的必要測試方法。

At Google, we have a specialized engineering role of 『Test Engineer,』 and one of the things we look for in a good test engineer is the ability to outline a test strategy for our products.

在谷歌，我們有一個專門的工程角色『測試工程師』，我們在一個好的測試工程師身上尋找的東西之一就是能夠為我們的產品勾勒出一個測試策略。

### Functional Testing of One or More Interacting Binaries 一個或多個二進位檔案的功能測試

Tests of these type have the following characteristics:

- SUT: single-machine hermetic or cloud-deployed isolated
- Data: handcrafted
- Verification: assertions

此類試驗具有以下特點：

- SUT：單機密封或雲部署隔離
- 資料：手工製作
- 核查：斷言

As we have seen so far, unit tests are not capable of testing a complex system with true fidelity, simply because they are packaged in a different way than the real code is packaged. Many functional testing scenarios interact with a given binary differently than with classes inside that binary, and these functional tests require separate SUTs and thus are canonical, larger tests.

到目前為止，我們已經看到，單元測試無法以真正模擬地測試複雜的系統，僅僅是因為它們的打包方式與實際程式碼的打包方式不同。許多功能測試場景與給定二進位檔案的互動方式不同於與該二進位檔案中的類別的互動方式，這些功能測試需要單獨的SUT，因此是經典的大型測試。

Testing the interactions of multiple binaries is, unsurprisingly, even more complicated than testing a single binary. A common use case is within microservices environments when services are deployed as many separate binaries. In this case, a functional test can cover the real interactions between the binaries by bringing up an SUT composed of all the relevant binaries and by interacting with it through a published API.

毫不奇怪，測試多個二進位檔案的相互作用甚至比測試單個二進位檔案更復雜。一個常見的案例是在微服務環境中，當服務被部署為許多獨立的二進位檔案。在這種情況下，功能測試可以透過提出由所有相關二進位檔案組成的SUT，並透過發布的API與之互動，來覆蓋二進位檔案之間的真實互動。

### Browser and Device Testing 瀏覽器和裝置測試

Testing web UIs and mobile applications is a special case of functional testing of one or more interacting binaries. It is possible to unit test the underlying code, but for the end users, the public API is the application itself. Having tests that interact with the application as a third party through its frontend provides an extra layer of coverage.

測試web UI和行動應用程式是對一個或多個相互協作二進位檔案進行功能測試的特例。可以對底層程式碼進行單元測試，但對於最終使用者來說，公共API是應用程式本身。將測試作為第三方透過其前端與應用程式互動提供了額外的覆蓋層。

### Performance, Load, and Stress testing 效能、負載和壓力測試
Tests of these type have the following characteristics:

- SUT: cloud-deployed isolated
- Data: handcrafted or multiplexed from production
- Verification: diff (performance metrics)

此類試驗具有以下特點：

- SUT：雲部署隔離
- 資料：手工生成或從生產中多路傳輸
- 驗證：差異（效能指標）

Although it is possible to test a small unit in terms of performance, load, and stress, often such tests require sending simultaneous traffic to an external API. That definition implies that such tests are multithreaded tests that usually test at the scope of a binary under test. However, these tests are critical for ensuring that there is no degradation in performance between versions and that the system can handle expected spikes in traffic.

儘管可以在效能、負載和壓力方面測試較小的單元，但此類測試通常需要同時向外部API傳送通訊量。該定義意味著此類測試是多執行緒測試，通常在被測二進位檔案的範圍內進行測試。但是，這些測試對於確保版本之間的效能不會下降以及系統能夠處理預期的流量峰值至關重要。

As the scale of the load test grows, the scope of the input data also grows, and it eventually becomes difficult to generate the scale of load required to trigger bugs under load. Load and stress handling are 『highly emergent』 properties of a system; that is, these complex behaviors belong to the overall system but not the individual members. Therefore, it is important to make these tests look as close to production as possible. Each SUT requires resources akin to what production requires, and it becomes difficult to mitigate noise from the production topology.

隨著負載測試規模的增長，輸入資料的範圍也在增長，甚至很難在負載下生成觸發bug所需的負載規模。負載和壓力處理是系統的 "高度湧現"屬性；也就是說，這些複雜的行為屬於整個系統，而不是個別組成。因此，重要的是使這些測試看起來儘可能地接近生產。每個SUT所需的資源與生產所需的資源類似，因此很難緩解生產拓撲中的噪音。

One area of research for eliminating noise in performance tests is in modifying the deployment topology—how the various binaries are distributed across a network of machines. The machine running a binary can affect the performance characteristics; thus, if in a performance diff test, the base version runs on a fast machine (or one with a fast network) and the new version on a slow one, it can appear like a performance regression. This characteristic implies that the optimal deployment is to run both versions on the same machine. If a single machine cannot fit both versions of the binary, an alternative is to calibrate by performing multiple runs and removing peaks and valleys.

消除效能測試中的噪音的一個研究領域是修改部署拓撲結構——各種二進位檔案在機器網路中的分佈。執行二進位檔案的機器會影響效能屬性；因此，如果在效能差異測試中，基本版本在快速機器（或具有高速網路的機器）上執行，而新版本在慢速機器上執行，則可能會出現效能迴歸。此屬性意味著最佳部署是在同一台機器上執行兩個版本。如果一台機器無法同時安裝兩種版本的二進位檔案，另一種方法是透過執行多次執行並消除峰值和谷值來進行校準。

### Deployment Configuration Testing 部署設定測試

Tests of these type have the following characteristics:

- SUT: single-machine hermetic or cloud-deployed isolated
- Data: none
- Verification: assertions (doesn」t crash)

此類試驗具有以下特點：

- SUT：單機封閉或雲部署隔離
- 資料：無
- 驗證：斷言（不會崩潰）

Many times, it is not the code that is the source of defects but instead configuration: data files, databases, option definitions, and so on. Larger tests can test the integration of the SUT with its configuration files because these configuration files are read during the launch of the given binary.

很多時候，缺陷的根源不是程式碼，而是設定：資料檔案、資料庫、選項定義等等。較大的測試可以測試SUT與其設定檔案的整合，因為這些設定檔案是在給定二進位檔案啟動期間讀取的。

Such a test is really a smoke test of the SUT without needing much in the way of additional data or verification. If the SUT starts successfully, the test passes. If not, the test fails.

這種測試實際上是SUT的冒煙測試，不需要太多額外的資料或驗證。如果SUT成功啟動，則測試透過。否則，測試失敗。

### Exploratory Testing 探索性測試

Tests of these type have the following characteristics:

- SUT: production or shared staging
- Data: production or a known test universe
- Verification: manual

此類試驗具有以下特點：

- SUT：生產或共享預發
- 資料：生產或已知測試範圍
- 核查：手動

Exploratory testing[^2] is a form of manual testing that focuses not on looking for behavioral regressions by repeating known test flows, but on looking for questionable behavior by trying out new user scenarios. Trained users/testers interact with a product through its public APIs, looking for new paths through the system and for which behavior deviates from either expected or intuitive behavior, or if there are security vulnerabilities.

探索性測試是一種手動測試，它的重點不是透過重複已知的測試流程來尋找已知行為的迴歸測試，而是透過嘗試新的使用者場景來尋找有問題的行為。訓練有素的使用者/測試人員透過產品的公共API與產品互動，在系統中尋找新的路徑，尋找行為偏離預期或直觀行為的路徑，或者是否存在安全漏洞。

Exploratory testing is useful for both new and launched systems to uncover unanticipated behaviors and side effects. By having testers follow different reachable paths through the system, we can increase the system coverage and, when these testers identify bugs, capture new automated functional tests. In a sense, this is a bit like a manual 『fuzz testing』 version of functional integration testing.

探索性測試對於新系統和已發布系統都很有用，可以發現意外行為和副作用。透過讓測試人員在系統中遵循不同的可到達路徑，我們可以增加系統覆蓋率，並且當這些測試人員發現bug時，可以捕獲新的自動化功能測試。在某種意義上，這有點像功能整合測試的手動『模糊測試』版本。

>[^2]: James A. Whittaker, Exploratory Software Testing: Tips, Tricks, Tours, and Techniques to Guide Test Design(New York: Addison-Wesley Professional, 2009).
>
> 2     詹姆斯·惠塔克，探索性軟體測試： 提示， 詭計， 旅行，和技巧到指導測驗設計（紐約：Addison-Wesley Professional，2009年）。

#### Limitations 侷限性

Manual testing does not scale sublinearly; that is, it requires human time to perform the manual tests. Any defects found by exploratory tests should be replicated with an automated test that can run much more frequently.

手動測試無法進行亞線性擴充套件；也就是說，執行手動測試需要人工時間。透過探索性測試發現的任何缺陷都應該透過能夠更頻繁地執行的自動化測試進行複製。

#### Bug bashes 掃除bug

One common approach we use for manual exploratory testing is the [bug bash](https://oreil.ly/zRLyA). A team of engineers and related personnel (managers, product managers, test engineers, anyone with familiarity with the product) schedules a 『meeting,』 but at this session, everyone involved manually tests the product. There can be some published guidelines as to particular focus areas for the bug bash and/or starting points for using the system, but the goal is to provide enough interaction variety to document questionable product behaviors and outright bugs.

我們用於手動探索性測試的一種常見方法是bug大掃除。一組工程師和相關人員（經理、產品經理、測試工程師、熟悉產品的任何人）安排了一次『會議』，但在此情況下，所有相關人員都會手動測試產品。對於bug 大掃除的特定關注領域和/或使用系統的起點，可能會有一些已發布的指南，但目標是提供足夠的互動多樣性，以記錄有問題的產品行為和底層的bug。

### A/B Diff Regression Testing  A/B對比測試

Tests of these type have the following characteristics:

- SUT: two cloud-deployed isolated environments
- Data: usually multiplexed from production or sampled
- Verification: A/B diff comparison

此類試驗具有以下特點：

- SUT：兩個雲部署的隔離環境
- 資料：通常從生產或取樣中多路傳輸
- 驗證：A/B差異比較

Unit tests cover expected behavior paths for a small section of code. But it is impossible to predict many of the possible failure modes for a given publicly facing product. Additionally, as Hyrum」s Law states, the actual public API is not the declared one but all user-visible aspects of a product. Given those two properties, it is no surprise that A/B diff tests are possibly the most common form of larger testing at Google. This approach conceptually dates back to 1998. At Google, we have been running tests based on this model since 2001 for most of our products, starting with Ads, Search, and Maps.

單元測試覆蓋了一小部分程式碼的預期行為路徑。但是，對於給定的面向公眾的產品，預測多種可能的故障模式是不可行的。此外，正如海勒姆定律所指出的，實際的公共API不是宣告的API，而是一個產品的所有使用者可見的方面。鑑於這兩個屬性，A/B對比測試可能是谷歌最常見的大型測試形式，這並不奇怪。這種方法在概念上可以追溯到1998年。在谷歌，我們從2001年開始為我們的大多數產品進行基於這種模式的測試，從廣告、搜尋和地圖開始。

A/B diff tests operate by sending traffic to a public API and comparing the responses between old and new versions (especially during migrations). Any deviations in behavior must be reconciled as either anticipated or unanticipated (regressions). In this case, the SUT is composed of two sets of real binaries: one running at the candidate version and the other running at the base version. A third binary sends traffic and compares the results.

A/B對比測試透過向公共API傳送流量並比較新舊版本之間的回應（特別是在遷移期間）。任何行為上的偏差都必須作為預期的或未預期的（迴歸）進行調整。在這種情況下，SUT由兩組真實的二進位檔案組成：一個執行在候選版本，另一個執行在基本版本。第三個二進位程式傳送流量並比較結果。

There are other variants. We use A-A testing (comparing a system to itself) to identify nondeterministic behavior, noise, and flakiness, and to help remove those from A-B diffs. We also occasionally use A-B-C testing, comparing the last production version, the baseline build, and a pending change, to make it easy at one glance to see not only the impact of an immediate change, but also the accumulated impacts of what would be the next-to-release version.

還有其他的變體。我們使用A-A測試（將系統與自身進行比較）來識別非決定性行為、噪音和脆弱性，並幫助從A-B差異中去除這些東西。我們有時也會使用A-B-C測試，比較最後的生產版本、基線建立和一個待定的變化，以便一眼就能看出即時更改的影響，以及下一個發布版本的累積影響。

A/B diff tests are a cheap but automatable way to detect unanticipated side effects for any launched system.

A/B差異測試是一種低成本但可自動檢測任何已啟動系統意外副作用的方法。

#### Limitations  侷限性

Diff testing does introduce a few challenges to solve:

- *Approval*  
	Someone must understand the results enough to know whether any differences are expected. Unlike a typical test, it is not clear whether diffs are a good or bad thing (or whether the baseline version is actually even valid), and so there is often a manual step in the process.
- *Noise*  
	For a diff test, anything that introduces unanticipated noise into the results leads to more manual investigation of the results. It becomes necessary to remediate noise, and this is a large source of complexity in building a good diff test.
- *Coverage*  
	Generating enough useful traffic for a diff test can be a challenging problem. The test data must cover enough scenarios to identify corner-case differences, but it is difficult to manually curate such data.
- *Setup*  
	Configuring and maintaining one SUT is fairly challenging. Creating two at a time can double the complexity, especially if these share interdependencies.

對比測試確實帶來了一些需要解決的挑戰：

- *批准*  
	必須有人對結果有足夠的瞭解，才能知道是否會出現任何差異。與典型的測試不同，不清楚差異是好是壞（或者基線版本實際上是否有效），因此在這個過程中通常需要手動步驟。
- *噪音*  
	對於對比測試來說，任何在結果中引入意料之外的噪音都會導致對結果進行更多的手動查驗。有必要對噪聲進行補救，這也是建立一個好的對比測試的一個很大的複雜性來源。
- *覆蓋率*  
	為對比測試產生足夠的有用流量是一個具有挑戰性的問題。測試資料必須涵蓋足夠多的場景，以確定角落的差異，但很難手動管理這樣的資料。
- *設定*  
	設定和維護一個SUT是相當具有挑戰性的。一次建立兩個可以使複雜性加倍，特別是如果這些共享相互依賴關係。

### UAT

Tests of these type have the following characteristics:

- SUT: machine-hermetic or cloud-deployed isolated
- Data: handcrafted
- Verification: assertions

此類試驗具有以下特點：

- SUT：機器密封或雲部署隔離
- 資料：手工製作
- 核查：斷言

A key aspect of unit tests is that they are written by the developer writing the code under test. But that makes it quite likely that misunderstandings about the *intended* behavior of a product are reflected not only in the code, but also the unit tests. Such unit tests verify that code is 『Working as implemented』 instead of 『Working as intended.』

單元測試的一個關鍵方面是，它們是由編寫被測程式碼的開發人員編寫的。但是，這使得對產品的*預期*行為的誤解很可能不僅反映在程式碼中，而且也反映在單元測試中。這樣的單元測試驗證了程式碼是 "按實現工作 "而不是 "按預期工作"。

For cases in which there is either a specific end customer or a customer proxy (a customer committee or even a product manager), UATs are automated tests that exercise the product through public APIs to ensure the overall behavior for specific [user jour‐](https://oreil.ly/lOaOq) [neys ](https://oreil.ly/lOaOq)is as intended. Multiple public frameworks exist (e.g., Cucumber and RSpec) to make such tests writable/readable in a user-friendly language, often in the context of 『runnable specifications.』

對於有特定終端客戶或客戶代理（客戶委員會甚至產品經理）的情況，UAT是透過公共API執行產品的自動化測試，以確保特定[使用者旅程](https://oreil.ly/lOaOq)的總體行為符合預期。存在多個公共框架（例如，Cucumber和RSpec），使這種測試可以用使用者友好的語言寫/讀，通常是在 "可執行規範"的背景下。

Google does not actually do a lot of automated UAT and does not use specification languages very much. Many of Google」s products historically have been created by the software engineers themselves. There has been little need for runnable specification languages because those defining the intended product behavior are often fluent in the actual coding languages themselves.

谷歌實際上並沒有做很多自動化的UAT，也不怎麼使用規範語言。谷歌的許多產品在歷史上都是由軟體工程師自己建立的。幾乎不需要可執行的規範語言，因為那些定義預期產品行為的規範語言通常能夠流利地使用實際的編碼語言。

### Probers and Canary Analysis 探針和金絲雀分析

Tests of these type have the following characteristics:

- SUT: production
- Data: production
- Verification: assertions and A/B diff (of metrics)

此類試驗具有以下特點：

- SUT：生產
- 資料：生產
- 驗證：斷言和A/B差異（度量）

Probers and canary analysis are ways to ensure that the production environment itself is healthy. In these respects, they are a form of production monitoring, but they are structurally very similar to other large tests.

探針和金絲雀分析是確保生產環境本身健康的方法。在這些方面，它們是生產監控的一種形式，但在結構上與其他大型測試非常相似。

Probers are functional tests that run encoded assertions against the production environment. Usually these tests perform well-known and deterministic read-only actions so that the assertions hold even though the production data changes over time. For example, a prober might perform a Google search at [www.google.com ](http://www.google.com/)and verify that a result is returned, but not actually verify the contents of the result. In that respect, they are 『smoke tests』 of the production system, but they provide early detection of major issues.

Probers是功能測試，針對生產環境執行編碼的斷言。通常，這些測試執行眾所周知的和確定的只讀動作，這樣即使生產資料隨時間變化，斷言也能成立。例如，探針可能在 [www.google.com](http://www.google.com/) 執行谷歌搜尋，並驗證回傳的結果，但實際上並不驗證結果的內容。在這方面，它們是生產系統的 "冒煙測試"，但可以及早發現重大問題。

Canary analysis is similar, except that it focuses on when a release is being pushed to the production environment. If the release is staged over time, we can run both prober assertions targeting the upgraded (canary) services as well as compare health metrics of both the canary and baseline parts of production and make sure that they are not out of line.

金絲雀分析也是類似的，只不過它關注的是一個版本何時被推送到生產環境。如果發布是分階段進行的，我們可以同時執行針對升級（金絲雀）服務的探針斷言，以及比較生產中金絲雀和基線部分的健康指標，並確保它們沒有失衡。

Probers should be used in any live system. If the production rollout process includes a phase in which the binary is deployed to a limited subset of the production machines (a canary phase), canary analysis should be used during that procedure.

探針應該在任何實時系統中使用。如果生產推廣過程包括一個階段，其中二進位檔案被部署到生產機器的有限子集（一個金絲雀階段），則金絲雀分析應該在該過程中使用。

#### Limitations 侷限性

Any issues caught at this point in time (in production) are already affecting end users.

此時（生產中）發現的任何問題都已經影響到最終使用者。

If a prober performs a mutable (write) action, it will modify the state of production. This could lead to one of three outcomes: nondeterminism and failure of the assertions, failure of the ability to write in the future, or user-visible side effects.

如果探針執行可變（寫入）操作，它將修改生產狀態。這可能導致以下三種結果之一：不確定性和評估失敗、未來寫入能力失敗或使用者可見的副作用。

### Disaster Recovery and Chaos Engineering 故障恢復與混沌工程

Tests of these type have the following characteristics:

- SUT: production
- Data: production and user-crafted (fault injection)
- Verification: manual and A/B diff (metrics)

此類試驗具有以下特點：

- SUT：生產
- 資料：生產和使用者定製（故障注入）
- 驗證：手動和A/B對比（指標）

These test how well your systems will react to unexpected changes or failures.

這些測試將測試系統對意外更改或故障的反應。

For years, Google has run an annual war game called [DiRT 
](https://oreil.ly/17ffL)(Disaster Recovery Testing) during which faults are injected into our infrastructure at a nearly planetary scale. We simulate everything from datacenter fires to malicious attacks. In one memorable case, we simulated an earthquake that completely isolated our headquarters in Mountain View, California, from the rest of the company. Doing so exposed not only technical shortcomings but also revealed the challenge of running a company when all the key decision makers were unreachable.[^3]

多年來，谷歌每年都會舉辦一場名為『災難恢復測試』[DiRT](https://oreil.ly/17ffL)(Disaster Recovery Testing)的演練，在這場演練中，故障幾乎以全球規模注入我們的基礎設施。我們模擬了從資料中心火災到惡意攻擊的一切。在一個令人難忘的案例中，我們模擬了一場地震，將我們位於加州山景城的總部與公司其他部門完全隔離。這樣做不僅暴露了技術上的缺陷，也揭示了在所有關鍵決策者都無法聯絡到的情況下，管理公司的挑戰。

The impacts of DiRT tests require a lot of coordination across the company; by contrast, chaos engineering is more of a 『continuous testing』 for your technical infrastructure. [Made popular by Netflix](https://oreil.ly/BCwdM), chaos engineering involves writing programs that continuously introduce a background level of faults into your systems and seeing what happens. Some of the faults can be quite large, but in most cases, chaos testing tools are designed to restore functionality before things get out of hand. The goal of chaos engineering is to help teams break assumptions of stability and reliability and help them grapple with the challenges of building resiliency in. Today, teams at Google perform thousands of chaos tests each week using our own home-grown system called Catzilla.

DiRT測試的影響需要整個公司的大量協調；相比之下，混沌工程更像是對你的技術基礎設施的 "持續測試"。[由Netflix推廣](https://oreil.ly/BCwdM)，混沌工程包括編寫程式，在你的系統中不斷引入背景水平的故障，並觀察會發生什麼。有些故障可能相當大，但在大多數情況下，混沌測試工具旨在在事情失控之前恢復功能。混沌工程的目標是幫助團隊打破穩定性和可靠性的假設，幫助他們應對建立彈性的挑戰。今天，谷歌的團隊每週都會使用我們自己開發的名為Catzilla的系統進行數千次混沌測試。

These kinds of fault and negative tests make sense for live production systems that have enough theoretical fault tolerance to support them and for which the costs and risks of the tests themselves are affordable.

這些型別的故障和負面測試對於具有足夠理論容錯能力的實時生產系統是有意義的，並且測試本身的成本和風險是可以承受的。

> [^3]:	During this test, almost no one could get anything done, so many people gave up on work and went to one of our many cafes, and in doing so, we ended up creating a DDoS attack on our cafe teams!
>
> 3   在這次測試中，幾乎沒有人能完成任何事情，所以很多人放棄了工作，去了我們眾多咖啡館中的一家，在這樣做的過程中，我們最終對我們的咖啡館團隊發起了DDoS攻擊！

#### Limitations 侷限性

Any issues caught at this point in time (in production) are already affecting end users.

此時（生產中）發現的任何問題都已經影響到最終使用者。

DiRT is quite expensive to run, and therefore we run a coordinated exercise on an infrequent scale. When we create this level of outage, we actually cause pain and negatively impact employee performance.

DiRT的執行成本相當高，因此我們不經常進行協作演練。當我們製造這種程度的故障時，我們實際上造成了痛苦，並對員工的績效產生了負面影響。

If a prober performs a mutable (write) action, it will modify the state of production. This could lead to either nondeterminism and failure of the assertions, failure of the ability to write in the future, or user-visible side effects.

如果探針執行了一個可變（寫）的動作，它將修改生產的狀態。這可能導致非確定性和斷言的失敗，未來寫入能力的失敗，或使用者可見的副作用。

### User Evaluation 使用者評價

Tests of these type have the following characteristics:

- SUT: production
- Data: production
- Verification: manual and A/B diffs (of metrics)

此類試驗具有以下特點：

- SUT：生產
- 資料：生產
- 驗證：手動和A/B對比（度量）

Production-based testing makes it possible to collect a lot of data about user behavior. We have a few different ways to collect metrics about the popularity of and issues with upcoming features, which provides us with an alternative to UAT:

- *Dogfooding*  
	It」s possible using limited rollouts and experiments to make features in production available to a subset of users. We do this with our own staff sometimes (eat our own dogfood), and they give us valuable feedback in the real deployment environment.
- *Experimentation*  
	A new behavior is made available as an experiment to a subset of users without their knowing. Then, the experiment group is compared to the control group at an aggregate level in terms of some desired metric. For example, in YouTube, we had a limited experiment changing the way video upvotes worked (eliminating the downvote), and only a portion of the user base saw this change.
	This is a [massively important approach for Google](https://oreil.ly/OAvqF). One of the first stories a Noogler hears upon joining the company is about the time Google launched an experiment changing the background shading color for AdWords ads in Google Search and noticed a significant increase in ad clicks for users in the experimental group versus the control group.
- *Rater* *evaluation*  
	Human raters are presented with results for a given operation and choose which one is 『better』 and why. This feedback is then used to determine whether a given change is positive, neutral, or negative. For example, Google has historically used rater evaluation for search queries (we have published the guidelines we give our raters). In some cases, the feedback from this ratings data can help determine launch go/no-go for algorithm changes. Rater evaluation is critical for nondeterministic systems like machine learning systems for which there is no clear correct answer, only a notion of better or worse.
	

基於產品的測試可以收集大量關於使用者行為的資料。我們有幾種不同的方法來收集有關即將推出的功能的受歡迎程度和問題的指標，這為我們提供了UAT的替代方案：

- *吃自己的狗糧*  
	我們可以利用有限的推廣和實驗，將生產中的功能提供給一部分使用者使用。我們有時會和自己的員工一起這樣做（吃自己的狗糧），他們會在真實的部署環境中給我們提供寶貴的反饋。
- *實驗*  
	在使用者不知情的情況下，將一個新的行為作為一個實驗提供給一部分使用者。然後，將實驗組與控制組在某種期望的指標方面進行綜合比較。例如，在YouTube，我們做了一個有限的實驗，改變了影片加分的方式（取消了降分），只有一部分使用者看到了這個變化。
	這是一個[對谷歌來說非常重要的方法](https://oreil.ly/OAvqF)。Noogler在加入公司後聽到的第一個故事是關於谷歌推出了一個實驗，改變了谷歌搜尋中AdWords廣告的背景陰影顏色，並注意到實驗組的使用者與對照組相比，廣告點選量明顯增加。
- *評分員評價*  
	評分員會被告知某一特定操作的結果，並選擇哪一個 "更好"以及原因。然後，這種反饋被用來確定一個特定的變更是正面、中性還是負面的。例如，谷歌在歷史上一直使用評分員對搜尋查詢進行評估（我們已經公佈了我們給評員者的指導方針）。在某些情況下，來自該評級資料的反饋有助於確定演算法更改的啟動透過/不通過。評價員的評價對於像機器學習系統這樣的非確定性系統至關重要，因為這些系統沒有明確的正確答案，只有一個更好或更差的概念。

## Large Tests and the Developer Workflow  大型測試和開發人員工作流程

We」ve talked about what large tests are, why to have them, when to have them, and how much to have, but we have not said much about the who. Who writes the tests? Who runs the tests and investigates the failures? Who owns the tests? And how do we make this tolerable?

我們已經討論了什麼是大型測試，為什麼要做測試，什麼時候做，做多少測試，但我們還沒有說太多是誰的問題。誰來寫測試？誰來執行測試並調查故障？誰擁有這些測試？我們如何讓這一切變得可以忍受？

Although standard unit test infrastructure might not apply, it is still critical to integrate larger tests into the developer workflow. One way of doing this is to ensure that automated mechanisms for presubmit and post-submit execution exist, even if these are different mechanisms than the unit test ones. At Google, many of these large tests do not belong in TAP. They are nonhermetic, too flaky, and/or too resource intensive. But we still need to keep them from breaking or else they provide no signal and become too difficult to triage. What we do, then, is to have a separate post-submit continuous build for these. We also encourage running these tests presubmit, because that provides feedback directly to the author.

儘管標準的單元測試基礎設施可能不適用，但將大型測試整合到開發人員的工作流程中仍然是至關重要的。做到這一點的一個方法是確保存在預送出和送出後執行的自動化機制，即使這些機制與單元測試的機制不同。在谷歌，許多大型測試不屬於TAP。它們不封閉、太不穩定和/或資源密集。但是我們仍然需要防止它們被破壞，否則它們就不能提供任何訊號，並且變得太難處理了。那麼，我們所做的就是為這些測試建立一個單獨的送出後持續建立。我們也鼓勵在送出前執行這些測試，因為這樣可以直接向作者提供反饋。

A/B diff tests that require manual blessing of diffs can also be incorporated into such a workflow. For presubmit, it can be a code-review requirement to approve any diffs in the UI before approving the change. One such test we have files release-blocking bugs automatically if code is submitted with unresolved diffs.

需要手動批准的A/B對比測試也可以被納入這樣一個工作流程。對於預送出，在批准更改之前批准UI中的任何差異可能是程式碼審查要求。我們有一個這樣的測試，如果送出的程式碼有未解決的差異，就會自動歸檔阻斷發布的錯誤。

In some cases, tests are so large or painful that presubmit execution adds too much developer friction. These tests still run post-submit and are also run as part of the release process. The drawback to not running these presubmit is that the taint makes it into the monorepo and we need to identify the culprit change to roll it back. But we need to make the trade-off between developer pain and the incurred change latency and the reliability of the continuous build.

在某些情況下，測試是如此之大或痛苦，以至於送出前的執行增加了太多的開發者負擔。這些測試仍然在送出後執行，並且作為發布過程的一部分執行。不在送出前執行這些測試的缺點是，bug會進入monorepo，我們需要確定罪魁禍首的變化來回滾它。但我們需要在開發人員的痛苦和所產生的變更延遲與持續建立的可靠性之間做出權衡。

### Authoring Large Tests 編寫大型測試

Although the structure of large tests is fairly standard, there is still a challenge with creating such a test, especially if it is the first time someone on the team has done so.

雖然大型測試的結構是相當標準的，但建立這樣的測試仍然存在挑戰，特別是當團隊中有人第一次操作時。

The best way to make it possible to write such tests is to have clear libraries, documentation, and examples. Unit tests are easy to write because of native language support (JUnit was once esoteric but is now mainstream). We reuse these assertion libraries for functional integration tests, but we also have created over time libraries for interacting with SUTs, for running A/B diffs, for seeding test data, and for orchestrating test workflows.

要使寫這種測試成為可能，最好的辦法是有明確的函式庫、文件和例子。單元測試很容易寫，因為有本地語言的支援（JUnit曾經很深奧，但現在是主流）。我們重新使用這些斷言庫進行功能整合測試，但隨著時間的推移，我們也建立了與SUT互動的函式庫，用於執行A/B差異，用於播種測試資料，以及用於協調測試工作流。

Larger tests are more expensive to maintain, in both resources and human time, but not all large tests are created equal. One reason that A/B diff tests are popular is that they have less human cost in maintaining the verification step. Similarly, production SUTs have less maintenance cost than isolated hermetic SUTs. And because all of this authored infrastructure and code must be maintained, the cost savings can compound.

大型測試在資源和人力時間方面的維護成本較高，但不是所有的大型測試都是一樣的。A/B對比測試受歡迎的一個原因是，它們在維護驗證步驟方面的人力成本較低。同樣，生產型SUT的維護成本比隔離的封閉型SUT要低。而且，由於所有這些自創的基礎設施和程式碼都必須被維護，成本的節省可以是落地的。

However, this cost must be looked at holistically. If the cost of manually reconciling diffs or of supporting and safeguarding production testing outweighs the savings, it becomes ineffective.

然而，必須從整體上看待這一成本。如果手動協調差異或支援和保護生產測試的成本超過了節省的成本，那麼它將變得無效。

### Running Large Tests 進行大型測試

We mentioned above how our larger tests don」t fit in TAP and so we have alternate continuous builds and presubmits for them. One of the initial challenges for our engineers is how to even run nonstandard tests and how to iterate on them.

我們在上面提到，我們的大型測試不適合在TAP中進行，所以我們為它們準備了備用的持續建立和預送出。對我們的工程師來說，最初的挑戰之一是如何執行非標準的測試，以及如何對它們進行迭代。

As much as possible, we have tried to make our larger tests run in ways familiar for our engineers. Our presubmit infrastructure puts a common API in front of running both these tests and running TAP tests, and our code review infrastructure shows both sets of results. But many large tests are bespoke and thus need specific documentation for how to run them on demand. This can be a source of frustration for unfamiliar engineers.

我們儘可能地使我們的大型測試以工程師熟悉的方式運作。我們的預送出基礎設施在執行這些測試和執行TAP測試之前都提供了一個通用的API，我們的程式碼審查基礎設施顯示了這兩組結果。但許多大型測試是定製的，因此需要具體的文件來說明如何按需執行它們。對於不熟悉的工程師來說，這可能是一個令人沮喪的原因。

#### Speeding up tests  加快測試進度

Engineers don」t wait for slow tests. The slower a test is, the less frequently an engineer will run it, and the longer the wait after a failure until it is passing again.

工程師不會等待緩慢的測試。測試越慢，工程師執行測試的頻率就越低，失敗後等待測試再次透過的時間就越長。

The best way to speed up a test is often to reduce its scope or to split a large test into two smaller tests that can run in parallel. But there are some other tricks that you can do to speed up larger tests.

加速測試的最佳方法通常是縮小其範圍，或者將大型測試拆分為兩個可以並行執行的小型測試。但是，您還可以使用其他一些技巧來加速更大的測試。

Some naive tests will use time-based sleeps to wait for nondeterministic action to occur, and this is quite common in larger tests. However, these tests do not have thread limitations, and real production users want to wait as little as possible, so it is best for tests to react the way real production users would. Approaches include the following:

- Polling for a state transition repeatedly over a time window for an event to complete with a frequency closer to microseconds. You can combine this with a timeout value in case a test fails to reach a stable state.
- Implementing an event handler.
- Subscribing to a notification system for an event completion.

一些簡單的測試會使用基於時間延遲注入來等待非確定性的動作發生，這在大型測試中是很常見的。但是，這些測試沒有執行緒限制，並且實際生產使用者希望等待的時間儘可能少，因此最好讓測試以實際生產使用者的方式做出反應。方法包括：

- 在時間視窗內重複輪詢狀態轉換，以使事件以接近微秒的頻率完成。如果測試無法達到穩定狀態，你可以將其與超時值結合起來。
- 實現一個事件處理程式。
- 訂閱事件完成通知系統。

Note that tests that rely on sleeps and timeouts will all start failing when the fleet running those tests becomes overloaded, which spirals because those tests need to be rerun more often, increasing the load further.

請注意，當執行這些測試的負載變得超載時，依賴延時和超時的測試都會開始失敗，這是因為這些測試需要更頻繁地重新執行，進一步增加了負載。

*Lower internal system timeouts and delays*  
	A production system is usually configured assuming a distributed deployment topology, but an SUT might be deployed on a single machine (or at least a cluster of colocated machines). If there are hardcoded timeouts or (especially) sleep statements in the production code to account for production system delay, these should be made tunable and reduced when running tests.

*Optimize test build time*  
	One downside of our monorepo is that all of the dependencies for a large test are built and provided as inputs, but this might not be necessary for some larger tests. If the SUT is composed of a core part that is truly the focus of the test and some other necessary peer binary dependencies, it might be possible to use prebuilt versions of those other binaries at a known good version. Our build system (based on the monorepo) does not support this model easily, but the approach is actually more reflective of production in which different services release at different versions.

*更低的內部系統超時和延遲*。  
	生產系統通常採用分散式部署拓撲進行設定，但SUT可能部署在一台機器上（或至少是一個群集的機器）。如果在生產程式碼中存在硬編碼超時或（特別是）休眠語句來解釋生產系統延遲，則應在執行測試時使其可調並減少。

*最佳化測試建立時間*。  
	我們的monorepo的一個缺點是，大型測試的所有依賴項都是作為輸入建立和提供的，但對於一些大型測試來說，這可能不是必需的。如果SUT是由一個真正的測試重點的核心部分和其他一些必要的對等二進位依賴組成的，那麼可以在已知的良好版本中使用這些其他二進位檔案的預建立版本。我們的建立系統（基於monorepo）不容易支援這種模式，但該方法實際上更能反映不同服務以不同版本發布的生產。

#### Driving out flakiness  驅除鬆散性

Flakiness is bad enough for unit tests, but for larger tests, it can make them unusable. A team should view eliminating flakiness of such tests as a high priority. But how can flakiness be removed from such tests?

對於單元測試來說，鬆散性已經很糟糕了，但對於大型測試來說，它可能會使它們無法使用。一個團隊應該把消除這種測試的鬆散性視為一個高度優先事項。但是，如何才能從這些測試中消除鬆散性呢？

Minimizing flakiness starts with reducing the scope of the test—a hermetic SUT will not be at risk of the kinds of multiuser and real-world flakiness of production or a shared staging environment, and a single-machine hermetic SUT will not have the network and deployment flakiness issues of a distributed SUT. But you can mitigate other flakiness issues through test design and implementation and other techniques. In some cases, you will need to balance these with test speed.

最大限度地減少鬆散，首先要減少測試的範圍——封閉的SUT不會有生產或共享預發環境的各種多使用者和真實世界鬆散的風險，單機封閉的SUT不會有分散式SUT的網路和部署閃失問題。但是你可以透過測試設計和實施以及其他技術來減輕其他的鬆散性問題。在某些情況下，你需要平衡這些與測試速度。

Just as making tests reactive or event driven can speed them up, it can also remove flakiness. Timed sleeps require timeout maintenance, and these timeouts can be embedded in the test code. Increasing internal system timeouts can reduce flakiness, whereas reducing internal timeouts can lead to flakiness if the system behaves in a nondeterministic way. The key here is to identify a trade-off that defines both a tolerable system behavior for end users (e.g., our maximum allowable timeout is *n* seconds) but handles flaky test execution behaviors well.

正如使測試反應式或事件驅動可以加快它們的速度一樣，它也可以消除鬆散性。定時休眠需要超時維護，這些超時可以嵌入測試程式碼中。增加系統的內部超時可以減少鬆散性，而減少內部超時可以導致鬆散性，如果系統的行為是不確定的。這裡的關鍵是確定一個權衡（平衡），既要為終端使用者定義一個可容忍的系統行為（例如，我們允許的最大超時是*n*秒），但很好地處理了不穩定的測試執行行為。

A bigger problem with internal system timeouts is that exceeding them can lead to difficult errors to triage. A production system will often try to limit end-user exposure to catastrophic failure by handling possible internal system issues gracefully. For example, if Google cannot serve an ad in a given time limit, we don」t return a 500, we just don」t serve an ad. But this looks to a test runner as if the ad-serving code might be broken when there is just a flaky timeout issue. It」s important to make the failure mode obvious in this case and to make it easy to tune such internal timeouts for test scenarios.

內部系統超時的一個更大問題是，超過這些超時會導致難以分類的錯誤。生產系統通常會試圖透過優雅地方式處理可能的內部系統問題來限制終端使用者對災難性故障的暴露。例如，如果谷歌不能在給定的時間限制內提供廣告，我們不會回傳500，我們只是不提供廣告。但在測試執行人員看來，如果只是出現異常超時問題，廣告服務可能會被中斷。在這種情況下，重要的是使故障模式變得明顯，並使調整測試場景的此類內部超時變得容易

#### Making tests understandable  讓測試變得易懂

A specific case for which it can be difficult to integrate tests into the developer workflow is when those tests produce results that are unintelligible to the engineer running the tests. Even unit tests can produce some confusion—if my change breaks your test, it can be difficult to understand why if I am generally unfamiliar with your code—but for larger tests, such confusion can be insurmountable. Tests that are assertive must provide a clear pass/fail signal and must provide meaningful error output to help triage the source of failure. Tests that require human investigation, like A/B diff tests, require special handling to be meaningful or else risk being skipped during presubmit.

當這些測試產生的結果對執行測試的工程師來說是無法理解的時候，就很難將測試整合到開發者的工作流程中。即使是單元測試也會產生一些混亂——如果我的修改破壞了你的測試，如果我一般不熟悉你的程式碼，就很難理解為什麼，但對於大型測試，這種混亂可能是無法克服的。堅定的測試必須提供一個明確的透過/失敗訊號，並且必須提供有意義的錯誤輸出，以幫助分類失敗的原因。需要人工調查的測試，如A/B對比測試，需要特殊處理才能有意義，否則在預送出期間有被跳過的風險。

How does this work in practice? A good large test that fails should do the following:

- *Have a message that clearly identifies what the failure is*  
	The worst-case scenario is to have an error that just says 『Assertion failed』 and a stack trace. A good error anticipates the test runner」s unfamiliarity with the code and provides a message that gives context: 『In test_ReturnsOneFullPageOfSearchResultsForAPopularQuery, expected 10 search results but got 1.』 For a performance or A/B diff test that fails, there should be a clear explanation in the output of what is being measured and why the behavior is considered suspect.
- *Minimize the effort necessary to identify the root cause of  the discrepancy*  
	A stack trace is not useful for larger tests because the call chain can span multiple process boundaries. Instead, it」s necessary to produce a trace across the call chain or to invest in automation that can narrow down the culprit. The test should produce some kind of artifact to this effect. For example, [Dapper](https://oreil.ly/FXzbv) is a framework used by Google to associate a single request ID with all the requests in an RPC call chain, and all of the associated logs for that request can be correlated by that ID to facilitate tracing.
- *Provide support and contact information.*  
	It should be easy for the test runner to get help by making the owners and supporters of the test easy to contact.

這在實踐中是如何運作的？一個成功的大型測試應該從失敗中獲取到訊息，要做到以下幾點：

- *有一個明確指出失敗原因的訊息*  
	最壞的情況是有一個錯誤，只是說 "斷言失敗 "和一個堆疊跟蹤。一個好的錯誤能預見到測試執行者對程式碼的不熟悉，並提供一個訊息來說明背景。』in test_ReturnsOneFullPageOfSearchResultsForAPopularQuery中，預期有10個搜尋結果，但得到了1個。" 對於失敗的效能或A/B對比測試，在輸出中應該有一個明確的解釋，說明什麼是被測量的，為什麼該行為被認為是可疑的。
- *儘量減少識別差異的根本原因所需的努力*  
	堆疊跟蹤對較大的測試沒有用，因為呼叫鏈可能跨越多個程序邊界。相反，有必要在整個呼叫鏈中產生一個跟蹤，或者投資於能夠縮小罪魁禍首的自動化。測試應該產生某種工具來達到這個效果。例如，[Dapper](https://oreil.ly/FXzbv) 是谷歌使用的一個框架，將一個單一的請求ID與RPC呼叫鏈中的所有請求相關聯，該請求的所有相關日誌都可以透過該ID進行關聯，以方便追蹤。
- *提供支援和聯絡訊息*  
	透過使測試的所有者和支援者易於聯絡，測試執行者應該很容易獲得幫助。

#### Owning Large Tests  大型測試所有權 

Larger tests must have documented owners—engineers who can adequately review changes to the test and who can be counted on to provide support in the case of test failures. Without proper ownership, a test can fall victim to the following:

- It becomes more difficult for contributors to modify and update the test
- It takes longer to resolve test failures

大型測試必須有記錄的所有者——他們可以充分審查測試的變更，並且在測試失敗的情況下，可以依靠他們提供支援。沒有適當的所有權，測試可能成為以下情況的受害者：

- 參與者修改和更新測試變得更加困難
- 解決測試失敗需要更長的時間

And the test rots.

而且測試也會腐爛。

Integration tests of components within a particular project should be owned by the project lead. Feature-focused tests (tests that cover a particular business feature across a set of services) should be owned by a 『feature owner』; in some cases, this owner might be a software engineer responsible for the feature implementation end to end; in other cases it might be a product manager or a 『test engineer』 who owns the description of the business scenario. Whoever owns the test must be empowered to ensure its overall health and must have both the ability to support its maintenance and the incentives to do so.

特定專案中元件的整合測試應由專案負責人負責。以功能為中心的測試（覆蓋一組服務中特定業務功能的測試）應由『功能所有者』負責；在某些情況下，該所有者可能是負責端到端功能實現的軟體工程師；在其他情況下，可能是負責業務場景描述的產品經理或『測試工程師』。無論誰擁有該測試，都必須有權確保其整體健康，並且必須具備支援其維護的能力和這樣做的激勵。

It is possible to build automation around test owners if this information is recorded in a structured way. Some approaches that we use include the following:

- *Regular code ownership*  
	In many cases, a larger test is a standalone code artifact that lives in a particular location in our codebase. In that case, we can use the OWNERS ([Chapter 9](#_bookmark664)) information already present in the monorepo to hint to automation that the owner(s) of a particular test are the owners of the test code.
- *Per-test* *annotations*  
	In some cases, multiple test methods can be added to a single test class or module, and each of these test methods can have a different feature owner. We use  per-language structured annotations to document the test owner in each of these cases so that if a particular test method fails, we can identify the owner to contact.

如果以結構化的方式記錄此訊息，則可以圍繞測試所有者建立自動化。我們使用的一些方法包括：

- *常規程式碼所有權*  
	在許多情況下，大型測試是一個獨立的程式碼構件，它位於程式碼庫中的特定位置。在這種情況下，我們可以使用monorepo中已經存在的所有者（第9章）訊息來提示自動化，特定測試的所有者是測試程式碼的所有者。

- *每個測試註解*  
	在某些情況下，可以將多個測試方法新增到單個測試類或模組中，並且這些測試方法中的每一個都可以有不同的屬性所有者。我們使用每種語言的結構化註解，用於記錄每種情況下的測試所有者，以便在特定測試方法失敗時，我們可以確定要聯絡的所有者。

## Conclusion 總結

A comprehensive test suite requires larger tests, both to ensure that tests match the fidelity of the system under test and to address issues that unit tests cannot adequately cover. Because such tests are necessarily more complex and slower to run, care must be taken to ensure such larger tests are properly owned, well maintained, and run when necessary (such as before deployments to production). Overall, such larger tests must still be made as small as possible (while still retaining fidelity) to avoid developer friction. A comprehensive test strategy that identifies the risks of a system, and the larger tests that address them, is necessary for most software projects.

一個全面的測試套件需要大型測試，既要確保測試與被測系統的模擬度相對應，又要解決單元測試不能充分覆蓋的問題。因為這樣的測試必然更復雜，執行速度更慢，所以必須注意確保這樣的大型測試是正確的，良好的維護，並在必要時運行（例如在部署到生產之前）。總的來說，這種大型測試仍然必須儘可能的小（同時仍然保留模擬度），以避免開發人員的阻力。一個全面的測試策略，確定系統的風險，以及解決這些風險的大型測試，對大多數軟體專案來說是必要的。

## TL;DRs  內容提要

- Larger tests cover things unit tests cannot.
- Large tests are composed of a System Under Test, Data, Action, and Verification.
- A good design includes a test strategy that identifies risks and larger tests that mitigate them.
- Extra effort must be made with larger tests to keep them from creating friction in the developer workflow.

- 大型測試涵蓋了單元測試不能涵蓋的內容。
- 大型測試是由被測系統、資料、操作和驗證組成。
- 良好的設計包括識別風險的測試策略和緩解風險的大型測試。
- 必須對大型測試做出額外的努力，以防止它們在開發者的工作流程中產生阻力。

