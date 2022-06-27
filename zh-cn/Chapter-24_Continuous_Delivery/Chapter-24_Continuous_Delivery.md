
**CHAPTER 24**

# Continuous Delivery

# 第二十四章 持續交付

                        Written by adha Narayan, Bobbi Jones, Sheri Shipe, and David Owens Edited by Lisa Carey

Given how quickly and unpredictably the technology landscape shifts, the competitive advantage for any product lies in its ability to quickly go to market. An organization’s velocity is a critical factor in its ability to compete with other players, maintain product and service quality, or adapt to new regulation. This velocity is bottlenecked by the time to deployment. Deployment doesn’t just happen once at initial launch. There is a saying among educators that no lesson plan survives its first contact with the student body. In much the same way, no software is perfect at first launch, and the only guarantee is that you’ll have to update it. Quickly.

鑑於技術領域的變化是如此之快且不可預測，任何產品的競爭優勢都在於其快速進入市場的能力。一個組織的速度是其與其他參與者競爭、保持產品和服務品質或適應新法規能力的關鍵因素。這種速度受到部署時間的瓶頸制約。部署不會在初始啟動時只發生一次。教育家們有一種說法，沒有一個教案能在第一次與學生接觸後倖存下來。同樣，沒有一款軟體在第一次發佈時是完美的，唯一的保證就是你必須更新它。迅速地

The long-term life cycle of a software product involves rapid exploration of new ideas, rapid responses to landscape shifts or user issues, and enabling developer velocity at scale. From Eric Raymond’s The Cathedral and the Bazaar to Eric Reis’ The Lean Startup, the key to any organization’s long-term success has always been in its ability to get ideas executed and into users’ hands as quickly as possible and reacting quickly to their feedback. Martin Fowler, in his book Continuous Delivery (aka CD), points out that “The biggest risk to any software effort is that you end up building something that isn’t useful. The earlier and more frequently you get working software in front of real users, the quicker you get feedback to find out how valuable it really is.”

軟體產品的長期生命週期包括快速探索新想法、快速響應環境變化或使用者問題，以及實現大規模開發速度。從埃裡克·雷蒙德（Eric Raymond）的《大教堂與集市》（The Cathedral and The Bazaar）到埃裡克·賴斯（Eric Reis）的《精益創業》（The Lean Startup），任何組織長期成功的關鍵始終在於其能夠儘快將想法付諸實施並交到使用者手中，並對他們的反饋做出快速反應。馬丁·福勒（Martin Fowler）在其著作《持續交付》（Continuous Delivery，又名CD）中指出，“任何軟體工作的最大風險是，你最終建立的東西並不實用。你越早、越頻繁地將工作中的軟體展現在真正的使用者面前，你就能越快地得到反饋，發現它到底有多大價值。”

Work that stays in progress for a long time before delivering user value is high risk and high cost, and can even be a drain on morale. At Google, we strive to release early and often, or “launch and iterate,” to enable teams to see the impact of their work quickly and to adapt faster to a shifting market. The value of code is not realized at the time of submission but when features are available to your users. Reducing the time between “code complete” and user feedback minimizes the cost of work that is in progress.

在交付使用者價值之前進行很長時間的工作是高風險和高成本的，甚至可能會消耗士氣。在 Google ，我們努力做到早期和經常發佈，或者說 "發佈和迭代"，以使團隊能夠迅速看到他們工作的影響，並更快地適應不斷變化的市場。程式碼的價值不是在提交時實現的，而是在你的使用者可以使用的功能時實現的。縮短 "程式碼完成 "和使用者反饋之間的時間，可以將正在進行中的工作的成本降到最低。

	You get extraordinary outcomes by realizing that the launch *never lands* but that it begins a learning cycle where you then fix the next most important thing, measure how it went, fix the next thing, etc.—and it is *never complete*.
	
	—David Weekly, Former Google product manager
	
	當你意識到發射從未著陸，但它開始了一個學習週期，然後你修復下一個最重要的事情，衡量它如何進行，修復下一個事情，等等——而且它永遠不會完成。
	-David Weekly，前 Google 產品經理

At Google, the practices we describe in this book allow hundreds (or in some cases thousands) of engineers to quickly troubleshoot problems, to independently work on new features without worrying about the release, and to understand the effectiveness of new features through A/B experimentation. This chapter focuses on the key levers of rapid innovation, including managing risk, enabling developer velocity at scale, and understanding the cost and value trade-off of each feature you launch.

在 Google ，我們在本書中描述的做法使數百名（或在某些情況下數千名）工程師能夠快速排除問題，獨立完成新功能而不必擔心發佈問題，並透過A/B實驗瞭解新功能的有效性。本章重點關注快速創新的關鍵槓桿，包括管理風險、實現大規模的開發者速度，以及瞭解你推出的每個功能的成本和價值權衡。

## Idioms of Continuous Delivery at Google  Google 持續交付的習慣用法

A core tenet of Continuous Delivery (CD) as well as of Agile methodology is that over time, smaller batches of changes result in higher quality; in other words, *faster is safer*. This can seem deeply controversial to teams at first glance, especially if the prerequisites for setting up CD—for example, Continuous Integration (CI) and testing— are not yet in place. Because it might take a while for all teams to realize the ideal of CD, we focus on developing various aspects that deliver value independently en route to the end goal. Here are some of these:

*Agility*

​	Release frequently and in small batches

*Automation*

​	Reduce or remove repetitive overhead of frequent releases

*Isolation*

​	Strive for modular architecture to isolate changes and make troubleshooting easier

*Reliability*

​	Measure key health indicators like crashes or latency and keep improving them

*Data-driven* *decision* *making*

​	Use A/B testing on health metrics to ensure quality

*Phased* *rollout*

​	Roll out changes to a few users before shipping to everyone

持續交付（CD）以及敏捷方法論的一個核心原則是，隨著時間的推移，小批量的變更會帶來更高的品質；換句話說，越快越安全。乍一看，這似乎對團隊有很大的爭議，尤其是當建立CD的前提條件--例如，持續整合（CI）和測試--還沒有到位的時候。因為所有團隊可能需要一段時間才能實現CD的理想，所以我們將重點放在開發能夠在實現最終目標的過程中獨立交付價值的各個方面。下面是其中的一些：

*敏捷性*

​	頻繁地、小批量地發佈。
*自動化*

​	減少或消除頻繁發佈的重複性開銷。
*隔離性*

​	努力實現模組化體系結構，以隔離更改並使故障排除更加容易。
*可靠性*

​	衡量關鍵的健康指標，如崩潰或延遲，並不斷改善它們。
*資料驅動的決策*

​	在健康指標上使用A/B測試以確保品質。
*分階段推出*

​	在向所有人傳送之前，先在少數使用者中推廣變更。

At first, releasing new versions of software frequently might seem risky. As your userbase grows, you might fear the backlash from angry users if there are any bugs that you didn’t catch in testing, and you might quite simply have too much new code in your product to test exhaustively. But this is precisely where CD can help. Ideally, there are so few changes between one release and the next that troubleshooting issues is trivial. In the limit, with CD, every change goes through the QA pipeline and is automatically deployed into production. This is often not a practical reality for many teams, and so there is often work of culture change toward CD as an intermediate step, during which teams can build their readiness to deploy at any time without actually doing so, building up their confidence to release more frequently in the future.

 起初，頻繁發佈新版本的軟體可能看起來很冒險。隨著使用者群的增長，你可能會擔心如果有任何你在測試中沒有發現的錯誤，你會受到憤怒的使用者的反擊，或許你可能在產品中包含太多的新程式碼，無法詳盡地測試。但這恰恰是CD可以幫助的地方。理想情況下，一個版本和下一個版本之間的變化非常少，排除問題是非常簡單的。在極限情況下，有了CD，每個變化都會透過QA管道，並自動部署到生產中。對於許多團隊來說，這通常不是一個實際的現實，因此，作為中間步驟，通常會有文化變革工作，在這一過程中，團隊可以建立隨時部署的準備，而不必實際這樣做，從而建立信心，在未來更頻繁地發佈。

## Velocity Is a Team Sport: How to Break Up a Deployment into Manageable Pieces   速度是一項團隊運動：如何將部署工作分解成可管理的部分

When a team is small, changes come into a codebase at a certain rate. We’ve seen an antipattern emerge as a team grows over time or splits into subteams: a subteam branches off its code to avoid stepping on anyone’s feet, but then struggles, later, with integration and culprit finding. At Google, we prefer that teams continue to develop at head in the shared codebase and set up CI testing, automatic rollbacks, and culprit finding to identify issues quickly. This is discussed at length in [Chapter 23](#_bookmark2022).

當一個團隊很小的時候，變化以一定的速度進入一個程式碼函式庫。我們看到，隨著時間的推移，一個團隊的成長或分裂成子團隊，會出現一種反模式：一個子團隊將其程式碼分支，以避免踩到其他團隊的腳，但後來卻在整合和尋找罪魁禍首方面陷入困境。在 Google ，我們更傾向於團隊繼續在共享程式碼函式庫中進行開發，並設定CI測試、自動回滾和故障查詢，以快速識別問題。這在第23章中有詳細的討論。

One of our codebases, YouTube, is a large, monolithic Python application. The release process is laborious, with Build Cops, release managers, and other volunteers. Almost every release has multiple cherry-picked changes and respins. There is also a 50-hour manual regression testing cycle run by a remote QA team on every release. When the operational cost of a release is this high, a cycle begins to develop in which you wait to push out your release until you’re able to test it a bit more. Meanwhile, someone wants to add just one more feature that’s almost ready, and pretty soon you have yourself a release process that’s laborious, error prone, and slow. Worst of all, the experts who did the release last time are burned out and have left the team, and now nobody even knows how to troubleshoot those strange crashes that happen when you try to release an update, leaving you panicky at the very thought of pushing that button.

我們的一個程式碼函式庫，YouTube，是一個大型的、單體的Python應用程式。發佈過程很費勁，有Build Cops、發佈經理和其他志願者。乎每個版本都有多個精心挑選的更改和響應。每個版本還有一個由遠端QA團隊執行的50小時手工迴歸測試周期。當一個發佈的操作成本如此之高時，就會形成一個迴圈，在這個迴圈中，你會等待推送你的版本，直到你能夠對其進行更多的測試。與此同時，有人想再增加一個幾乎已經準備好的功能，很快你就有了一個費力、容易出錯和緩慢的發佈過程。最糟糕的是，上次做發佈工作的專家已經精疲力盡，離開了團隊，現在甚至沒有人知道如何解決那些當你試圖發佈更新時發生的奇怪崩潰，讓你一想到要按下那個按鈕就感到恐慌。

If your releases are costly and sometimes risky, the *instinct* is to slow down your release cadence and increase your stability period. However, this only provides short- term stability gains, and over time it slows velocity and frustrates teams and users. The *answer* is to reduce cost, increase discipline, and make the risks more incremental, but it is critical to resist the obvious operational fixes and invest in long-term architectural changes. The obvious operational fixes to this problem lead to a few traditional approaches: reverting to a traditional planning model that leaves little room for learning or iteration, adding more governance and oversight to the development process, and implementing risk reviews or rewarding low-risk (and often low-value) features.

如果你的發佈是昂貴的，有時是有風險的，那麼*本能*的反應是放慢你的發佈節奏，增加你的穩定期。然而，這隻能提供短期的穩定性收益，隨著時間的推移，它會減慢速度，使團隊和使用者感到沮喪。答案是降低成本，提高紀律性，使風險更多的增加，但關鍵是要抵制明顯的操作修復，投資於長期的架構變化。對這個問題的明顯的操作性修正導致了一些傳統的方法：恢復到傳統的計劃模式，為學習或迭代留下很少的空間，為開發過程增加更多的治理和監督，以及實施風險審查或獎勵低風險（通常是低價值）的功能。

The investment with the best return, though, is migrating to a microservice architecture, which can empower a large product team with the ability to remain scrappy and innovative while simultaneously reducing risk. In some cases, at Google, the answer has been to rewrite an application from scratch rather than simply migrating it, establishing the desired modularity into the new architecture. Although either of these options can take months and is likely painful in the short term, the value gained in terms of operational cost and cognitive simplicity will pay off over an application’s lifespan of years.

不過，回報率最高的投資是遷移到微服務架構，這可以使一個大型產品團隊有能力保持活力和創新，同時降低風險。在某些情況下，在 Google ，答案是從頭開始重寫一個應用程式，而不是簡單地遷移它，在新的架構中建立所需的模組化。儘管這兩種選擇都需要幾個月的時間，而且在短期內可能是痛苦的，但在運營成本和認知的簡單性方面獲得的價值將在應用程式多年的生命週期中得到回報。

## Evaluating Changes in Isolation: Flag-Guarding Features 評估隔離中的更改：標誌保護功能

A key to reliable continuous releases is to make sure engineers “flag guard” *all changes*. As a product grows, there will be multiple features under various stages of development coexisting in a binary. Flag guarding can be used to control the inclusion or expression of feature code in the product on a feature-by-feature basis and can be expressed differently for release and development builds. A feature flag disabled for a build should allow build tools to strip the feature from the build if the language permits it. For instance, a stable feature that has already shipped to customers might be enabled for both development and release builds. A feature under development might be enabled only for development, protecting users from an unfinished feature. New feature code lives in the binary alongside the old codepath—both can run, but the new code is guarded by a flag. If the new code works, you can remove the old codepath and launch the feature fully in a subsequent release. If there’s a problem, the flag value can be updated independently from the binary release via a dynamic config update.

可靠的連續發佈的關鍵是確保工程師“保護”所有更改。隨著產品的發展，在不同的開發階段，將有多種功能以二進位制形式共存。標誌保護可用於控制產品中功能程式碼的包含或表達，以功能為基礎，並可在發佈和開發版本中以不同方式表達。如果區域網允許，為建構禁用的功能標誌應允許建構工具從建構中剝離該功能。例如，一個已經提供給客戶的穩定特性可能會在開發版本和發佈版本中啟用。正在開發的功能可能僅為開發而啟用，從而保護使用者不受未完成功能的影響。新的特性程式碼與舊的程式碼路徑一起存在於二進位制檔案中，兩者都可以執行，但新程式碼由一個標誌保護。如果新程式碼有效，您可以刪除舊程式碼路徑，並在後續版本中完全啟動該功能。如果出現問題，可以透過動態配置更新獨立於二進位制版本更新標誌值。

In the old world of binary releases, we had to time press releases closely with our binary rollouts. We had to have a successful rollout before a press release about new functionality or a new feature could be issued. This meant that the feature would be out in the wild before it was announced, and the risk of it being discovered ahead of time was very real.

在過去的二進位制發佈的世界裡，我們必須將新聞發佈時間與二進位制發佈時間緊密配合。在發佈關於新功能或新功能的新聞稿之前，我們必須進行成功的展示。這意味著該功能將在發佈之前就已經公開，提前被發現的風險是非常現實的。

This is where the beauty of the flag guard comes to play. If the new code has a flag, the flag can be updated to turn your feature on immediately before the press release, thus minimizing the risk of leaking a feature. Note that flag-guarded code is not a *perfect* safety net for truly sensitive features. Code can still be scraped and analyzed if it’s not well obfuscated, and not all features can be hidden behind flags without adding a lot of complexity. Moreover, even flag configuration changes must be rolled out with care. Turning on a flag for 100% of your users all at once is not a great idea, so a configuration service that manages safe configuration rollouts is a good investment. Nevertheless, the level of control and the ability to decouple the destiny of a particular feature from the overall product release are powerful levers for long-term sustainability of the application.

這就是標誌的魅力所在。如果新的程式碼有一個標誌，標誌可以被更新，在新聞發佈前立即開啟你的功能，從而最大限度地減少了功能洩露的風險。請注意，對於真正敏感的功能，有標誌的程式碼並不是一個完美的安全網。如果程式碼沒有被很好地混淆，它仍然可以被抓取和分析，而且不是所有的功能都可以隱藏在標誌後面而不增加很多複雜性。此外，即使是標誌配置的改變，也必須謹慎地推出。一次性為100%的使用者開啟一個標誌並不是一個好主意，所以一個能管理安全配置推出的配置服務是一個很好的投資。儘管如此，控制水平和將特定功能的命運與整個產品發佈脫鉤的能力是應用程式長期可持續性的有力槓桿。

## Striving for Agility: Setting Up a Release Train 為敏捷而奮鬥：建立一個發佈序列

Google’s Search binary is its first and oldest. Large and complicated, its codebase can be tied back to Google’s origin—a search through our codebase can still find code written at least as far back as 2003, often earlier. When smartphones began to take off, feature after mobile feature was shoehorned into a hairball of code written primarily for server deployment. Even though the Search experience was becoming more vibrant and interactive, deploying a viable build became more and more difficult. At one point, we were releasing the Search binary into production only once per week, and even hitting that target was rare and often based on luck.

谷搜尋是其第一個也是最古老的二進位制檔案。它的程式碼函式庫龐大而複雜，可以追溯到 Google 的起源--在我們的程式碼函式庫中搜索，仍然可以找到至少早在2003年編寫的程式碼，通常更早。當智慧手機開始使用時，一個接一個的移動功能被塞進了一大堆主要為伺服器部署而編寫的程式碼中。儘管搜尋體驗變得更加生動和互動，部署一個可行的建構變得越來越困難。有一次，我們每週只發布一次搜尋二進位制檔案到生產中，而即使達到這個目標也是很難得的，而且往往要靠運氣。

When one of our contributing authors, Sheri Shipe, took on the project of increasing our release velocity in Search, each release cycle was taking a group of engineers days to complete. They built the binary, integrated data, and then began testing. Each bug had to be manually triaged to make sure it wouldn’t impact Search quality, the user experience (UX), and/or revenue. This process was grueling and time consuming and did not scale to the volume or rate of change. As a result, a developer could never know when their feature was going to be released into production. This made timing press releases and public launches challenging.

當我們的貢獻作者之一Sheri Shipe承擔了提高搜尋發佈速度的專案時，每個發佈週期都需要一組工程師幾天才能完成。他們建構了二進位制整合資料，然後開始測試。每一個bug都必須進行手動分類，以確保它不會影響搜尋品質、使用者體驗（UX）和/或收入。這一過程既費時又費力，而且不能適應變化的數量和速度。因此，開發人員永遠不可能知道他們的特性將在何時發佈到生產環境中。這使得新聞發佈和公開發布的時間安排具有挑戰性。

Releases don’t happen in a vacuum, and having reliable releases makes the dependent factors easier to synchronize. Over the course of several years, a dedicated group of engineers implemented a continuous release process, which streamlined everything about sending a Search binary into the world. We automated what we could, set deadlines for submitting features, and simplified the integration of plug-ins and data into the binary. We could now consistently release a new Search binary into production every other day.

發佈不是在真空中發生的，擁有可靠的發佈使得依賴性因素更容易同步。在幾年的時間裡，一個專門的工程師小組實施了一個持續的發佈過程，它簡化了關於向世界傳送搜尋二進位制檔案的所有工作。我們把我們能做的事情自動化，為提交功能設定最後期限，並簡化外掛和資料到二進位制檔案的整合。我們現在可以每隔一天發佈一個新的搜尋二進位制檔案。

What were the trade-offs we made to get predictability in our release cycle? They narrow down to two main ideas we baked into the system.

為了在發佈週期中獲得可預測性，我們做了哪些權衡？他們把我們融入系統的兩個主要想法歸納了下來。

### No Binary Is Perfect 沒有完美的二進位制包

The first is that *no binary is perfect*, especially for builds that are incorporating the work of tens or hundreds of developers independently developing dozens of major features. Even though it’s impossible to fix every bug, we constantly need to weigh questions such as: If a line has been moved two pixels to the left, will it affect an ad display and potential revenue? What if the shade of a box has been altered slightly? Will it make it difficult for visually impaired users to read the text? The rest of this book is arguably about minimizing the set of unintended outcomes for a release, but in the end we must admit that software is fundamentally complex. There is no perfect binary—decisions and trade-offs have to be made every time a new change is released into production. Key performance indicator metrics with clear thresholds allow features to launch even if they aren’t perfect[1](#_bookmark2117) and can also create clarity in otherwise contentious launch decisions.

首先，*沒有一個二進位制包是完美的*，，尤其是對於包含數十個或數百個獨立開發幾十個主要功能的開發人員的工作的建構。儘管不可能修復每個bug，但我們需要不斷權衡這樣的問題：如果一條線向左移動了兩個畫素，它會影響廣告顯示和潛在收入嗎？如果盒子的顏色稍微改變了怎麼辦？這是否會讓視障使用者難以閱讀文字？本書的其餘部分可以說是關於最小化發佈的一系列意外結果，但最終我們必須承認軟體從根本上來說是複雜的。沒有完美的二進位制包--每當有新的變化發佈到生產中時，就必須做出決定和權衡。具有明確閾值的關鍵效能指標允許功能在不完美的情況下推出，也可以在其他有爭議的發佈決策中創造清晰的思路。

One bug involved a rare dialect spoken on only one island in the Philippines. If a user asked a search question in this dialect, instead of an answer to their question, they would get a blank web page. We had to determine whether the cost of fixing this bug was worth delaying the release of a major new feature.

其中一個bug涉及一種罕見的方言，這種方言只在菲律賓的一個島嶼上使用。如果使用者用這種方言問搜尋問題，而不是回答他們的問題，他們會得到一個空白網頁。我們必須確定修復這個bug的成本是否值得推遲發佈一個主要的新特性。

We ran from office to office trying to determine how many people actually spoke this language, if it happened every time a user searched in this language, and whether these folks even used Google on a regular basis. Every quality engineer we spoke with deferred us to a more senior person. Finally, data in hand, we put the question to Search’s senior vice president. Should we delay a critical release to fix a bug that affected only a very small Philippine island? It turns out that no matter how small your island, you should get reliable and accurate search results: we delayed the release and fixed the bug.

我們從一個辦公室跑到另一個辦公室，試圖確定究竟有多少人講這種語言，是否每次使用者用這種語言搜尋時都會出現這種情況，以及這些人是否經常使用 Google 。每個與我們交談的品質工程師都把我們推給更高級別的人。最後，資料在手，我們把問題交給了搜尋部的高階副總裁。我們是否應該推遲一個重要的版本來修復一個只影響到菲律賓一個很小的島嶼的錯誤？事實證明，無論你的島有多小，你都應該得到可靠和準確的搜尋結果：我們推遲了發佈，並修復了這個錯誤。

### Meet Your Release Deadline 滿足您的發佈期限

The second idea is that *if you’re late for the release train, it will leave without you*. There’s something to be said for the adage, “deadlines are certain, life is not.” At some point in the release timeline, you must put a stake in the ground and turn away developers and their new features. Generally speaking, no amount of pleading or begging will get a feature into today’s release after the deadline has passed.

第二個想法是，*如果你趕不上發佈列車，它就會丟下你離開*。有一句格言值得一提，“最後期限是確定的，生活不是確定的。”在發佈時間表的某個時刻，你必須立木取信，拒絕開發人員及其新功能。一般來說，在截止日期過後，無論多少懇求或乞求都不會在今天的版本中出現。

There is the *rare* exception. The situation usually goes like this. It’s late Friday evening and six software engineers come storming into the release manager’s cube in a panic. They have a contract with the NBA and finished the feature moments ago. But it must go live before the big game tomorrow. The release must stop and we must cherry- pick the feature into the binary or we’ll be in breach of contract! A bleary-eyed release engineer shakes their head and says it will take four hours to cut and test a new binary. It’s their kid’s birthday and they still need to pick up the balloons.

有一個*罕見的例外*。這種情況通常是這樣的。週五晚間，六名軟體工程師驚慌失措地衝進發布經理的辦公室。他們與NBA簽訂了合同，並在不久前完成了這個功能。但它必須在明天的大比賽之前上線。發佈必須停止，我們必須將該特性插入二進位制套件，否則我們將違反合同！"。一個目光呆滯的發佈工程師搖搖頭，說切割和測試一個新的二進位制檔案需要四個小時。今天是他們孩子的生日，他們還需要帶著氣球回家。

A world of regular releases means that if a developer misses the release train, they’ll be able to catch the next train in a matter of hours rather than days. This limits developer panic and greatly improves work–life balance for release engineers.

定期發佈的世界意味著，如果開發人員錯過了發版班車，他們將能夠在幾個小時而不是幾天內趕上下一班班車。這限制了開發人員的恐慌，並大大改善了發佈工程師的工作-生活平衡。

```
1  Remember the SRE “error-budget” formulation: perfection is rarely the best goal. Understand how much room for error is acceptable and how much of that budget has been spent recently and use that to adjust the trade-off between velocity and stability.
1 記住SRE的 "錯誤預算 "表述：完美很少是最佳目標。瞭解多少誤差空間是可以接受的，以及該預算最近花了多少，並利用這一點來調整速度和穩定性之間的權衡。
```

## Quality and User-Focus: Ship Only What Gets Used 品質和使用者關注點：只提供使用的產品

Bloat is an unfortunate side effect of most software development life cycles, and the more successful a product becomes, the more bloated its code base typically becomes. One downside of a speedy, efficient release train is that this bloat is often magnified and can manifest in challenges to the product team and even to the users. Especially if the software is delivered to the client, as in the case of mobile apps, this can mean the user’s device pays the cost in terms of space, download, and data costs, even for features they never use, whereas developers pay the cost of slower builds, complex deployments, and rare bugs. In this section, we’ll talk about how dynamic deployments allow you to ship only what is used, forcing necessary trade-offs between user value and feature cost. At Google, this often means staffing dedicated teams to improve the efficiency of the product on an ongoing basis.

膨脹是大多數軟體開發生命週期的一個不幸的副作用，產品越成功，其程式碼函式庫通常就越膨脹。快速、高效的發佈系列的一個缺點是，這種膨脹經常被放大，並可能表現為對產品團隊甚至使用者的挑戰。特別是如果軟體交付給客戶端（如移動應用程式），這可能意味著使用者的裝置要支付空間、下載和資料成本，即使是他們從未使用過的功能，而開發人員要支付建構速度較慢、部署複雜和罕見bug的成本。在本節中，我們將討論動態部署如何允許你僅發佈所使用的內容，從而在使用者價值和功能成本之間進行必要的權衡。在 Google ，這通常意味著配備專門的團隊，以不斷提高產品的效率。

Whereas some products are web-based and run on the cloud, many are client applications that use shared resources on a user’s device—a phone or tablet. This choice in itself showcases a trade-off between native apps that can be more performant and resilient to spotty connectivity, but also more difficult to update and more susceptible to platform-level issues. A common argument against frequent, continuous deployment for native apps is that users dislike frequent updates and must pay for the data cost and the disruption. There might be other limiting factors such as access to a network or a limit to the reboots required to percolate an update.

有些產品是基於網路並在雲上執行的，而許多產品是客戶端應用程式，使用使用者裝置上的共享資源--手機或平板電腦。這種選擇本身就展示了原生應用之間的權衡，原生應用可以有更高的效能，對不穩定的連線有彈性，但也更難更新，更容易受到平臺問題的影響。反對原生應用頻繁、持續部署的一個常見論點是，使用者不喜歡頻繁的更新，而且必須為資料成本和中斷付費。可能還有其他限制因素，如訪問網路或限制滲透更新所需的重新啟動。

Even though there is a trade-off to be made in terms of how frequently to update a product, the goal is to *have these choices be intentional*. With a smooth, well-running CD process, how often a viable release is *created* can be separated from how often a user *receives* it. You might achieve the goal of being able to deploy weekly, daily, or hourly, without actually doing so, and you should intentionally choose release processes in the context of your users’ specific needs and the larger organizational goals, and determine the staffing and tooling model that will best support the long-term sustainability of your product.

即使在更新產品的頻率方面需要做出權衡，但目標是*讓這些選擇是有意的*。有了一個平滑、執行良好的CD流程，建立一個可行版本的頻率可以與使用者收到它的頻率分開。你可能會實現每週、每天或每小時部署一次的目標，但實際上並沒有這樣做。你應該根據使用者的具體需求和更大的組織目標有意識地選擇發佈流程，並確定最能支援產品長期可持續性的人員配置和工具模型。

Earlier in the chapter, we talked about keeping your code modular. This allows for dynamic, configurable deployments that allow better utilization of constrained resources, such as the space on a user’s device. In the absence of this practice, every user must receive code they will never use to support translations they don’t need or architectures that were meant for other kinds of devices. Dynamic deployments allow apps to maintain small sizes while only shipping code to a device that brings its users value, and A/B experiments allow for intentional trade-offs between a feature’s cost and its value to users and your business.

在本章前面，我們討論了保持程式碼模組化。這允許動態、可配置的部署，以便更好地利用有限資源，例如使用者裝置上的空間。在沒有這種實踐的情況下，每個使用者都必須收到他們永遠不會使用的程式碼，以支援他們不需要的翻譯或用於其他型別裝置的架構。動態部署允許應用程式保持較小的尺寸，同時只將程式碼傳送給能為使用者帶來價值的裝置，而A/B實驗允許在功能的成本及其對使用者和企業的價值之間進行有意義的權衡。

There is an upfront cost to setting up these processes, and identifying and removing frictions that keep the frequency of releases lower than is desirable is a painstaking process. But the long-term wins in terms of risk management, developer velocity, and enabling rapid innovation are so high that these initial costs become worthwhile.

建立這些流程是有前期成本的，識別和消除使發佈頻率低於理想水平的阻力是一個艱苦的工作。但是，在風險管理、開發者速度和實現快速創新方面的長期勝利是如此之高，以至於這些初始成本是值得的。

## Shifting Left: Making Data-Driven Decisions Earlier 左移：提前做出資料驅動的決策

If you’re building for all users, you might have clients on smart screens, speakers, or Android and iOS phones and tablets, and your software may be flexible enough to allow users to customize their experience. Even if you’re building for only Android devices, the sheer diversity of the more than two billion Android devices can make the prospect of qualifying a release overwhelming. And with the pace of innovation, by the time someone reads this chapter, whole new categories of devices might have bloomed.

如果你是為所有使用者建立的，你可能在智慧螢幕、揚聲器或Android和iOS手機和平板電腦上有客戶，你的軟體可能足夠靈活，允許使用者訂製他們的體驗。即使你只為安卓裝置建構，超過20億的安卓裝置的多樣性也會使一個版本的場景變得不堪重負。隨著創新的步伐，當有人讀到這一章時，全新的裝置類別可能已經出現。

One of our release managers shared a piece of wisdom that turned the situation around when he said that the diversity of our client market was not a *problem*, but a *fact*. After we accepted that, we could switch our release qualification model in the following ways:

- If *comprehensive* testing is practically infeasible, aim for *representative* testing instead.

- Staged rollouts to slowly increasing percentages of the userbase allow for fast fixes.

- Automated A/B releases allow for statistically significant results proving a release’s quality, without tired humans needing to look at dashboards and make decisions.

我們的一位發佈經理分享了一條智慧，他說我們客戶市場的多樣性不是問題，而是事實，這扭轉了局面。在我們接受後，我們可以透過以下方式切換我們的發佈資格模型：

- 如果*全面*的測試實際上是不可行的，就以*代表性*的測試為目標。

- 分階段向用戶群中慢慢增加的百分比進行發佈，可以快速修復問題。

- 自動的A/B發佈允許統計學上有意義的結果來證明一個版本的品質，而無需疲憊的人去看儀表盤和做決定。

When it comes to developing for Android clients, Google apps use specialized testing tracks and staged rollouts to an increasing percentage of user traffic, carefully monitoring for issues in these channels. Because the Play Store offers unlimited testing tracks, we can also set up a QA team in each country in which we plan to launch, allowing for a global overnight turnaround in testing key features.

在為Android客戶端開發時， Google 應用程式使用專門的測試軌道和分階段推出，以增加使用者流量的百分比，仔細監控這些渠道中的問題。由於Play Store提供無限的測試軌道，我們還可以在我們計劃推出的每個國家/地區建立一個QA團隊，允許在全球範圍內一夜之間完成關鍵功能的測試。

One issue we noticed when doing deployments to Android was that we could expect a statistically significant change in user metrics *simply from pushing an update*. This meant that even if we made no changes to our product, pushing an update could affect device and user behavior in ways that were difficult to predict. As a result, although canarying the update to a small percentage of user traffic could give us good information about crashes or stability problems, it told us very little about whether the newer version of our app was in fact better than the older one.

我們在Android部署時注意到的一個問題是，僅僅透過推送更新，我們就可以預期使用者指標會發生統計上的顯著變化。這意味著，即使我們沒有對產品進行任何更改，推動更新也可能以難以預測的方式影響裝置和使用者行為。因此，儘管對一小部分使用者流量進行更新可以為我們提供關於崩潰或穩定性問題的良好資訊，但它幾乎沒有告訴我們更新版本的應用程式是否比舊版本更好。

Dan Siroker and Pete Koomen have already discussed the value of A/B testing[2](#_bookmark2127) your features, but at Google, some of our larger apps also A/B test their *deployments*. This means sending out two versions of the product: one that is the desired update, with the baseline being a placebo (your old version just gets shipped again). As the two versions roll out simultaneously to a large enough base of similar users, you can compare one release against the other to see whether the latest version of your software is in fact an improvement over the previous one. With a large enough userbase, you should be able to get statistically significant results within days, or even hours. An automated metrics pipeline can enable the fastest possible release by pushing forward a release to more traffic as soon as there is enough data to know that the guardrail metrics will not be affected.

Dan Siroker和Pete Koomen已經討論了A/B測試的價值，但在Google，我們的一些大型應用也對其*部署*進行A/B測試。這意味著傳送兩個版本的產品：一個是所需的更新，基線是一個安慰劑（你的舊版本只是被再次傳送）。當這兩個版本同時向足夠多的類似使用者推出時，你可以將一個版本與另一個版本進行比較，看看你的軟體的最新版本是否真的比以前的版本有所改進。有了足夠大的使用者群，你應該能夠在幾天內，甚至幾小時內得到統計學上的顯著結果。一個自動化的指標管道可以實現最快的發佈，只要有足夠的資料知道護欄指標不會受到影響，就可以將一個版本推到更多的流量。

Obviously, this method does not apply to every app and can be a lot of overhead when you don’t have a large enough userbase. In these cases, the recommended best practice is to aim for change-neutral releases. All new features are flag guarded so that the only change being tested during a rollout is the stability of the deployment itself.

顯然，這種方法並不適用於每個應用程式，當你沒有足夠大的使用者群時，可能會有很多開銷。在這種情況下，推薦的最佳做法是以變化中立的發佈為目標。所有的新功能都有標誌保護，這樣在發佈過程中測試的唯一變化就是部署本身的穩定性。 

```
2  Dan Siroker and Pete Koomen, *A/B Testing: The Most Powerful Way to Turn Clicks Into Customers* (Hoboken: Wiley, 2013).
2   Dan Siroker和Pete Koomen，《A/B測試：將點選轉化為客戶的最有效方式》（Hoboken:Wiley，2013）。
```

## Changing Team Culture: Building Discipline into Deployment 改變團隊文化：在部署中建立規則

Although “Always Be Deploying” helps address several issues affecting developer velocity, there are also certain practices that address issues of scale. The initial team launching a product can be fewer than 10 people, each taking turns at deployment and production-monitoring responsibilities. Over time, your team might grow to hundreds of people, with subteams responsible for specific features. As this happens and the organization scales up, the number of changes in each deployment and the amount of risk in each release attempt is increasing superlinearly. Each release contains months of sweat and tears. Making the release successful becomes a high-touch and labor-intensive effort. Developers can often be caught trying to decide which is worse: abandoning a release that contains a quarter’s worth of new features and bug fixes, or pushing out a release without confidence in its quality.

儘管“始終部署”有助於解決影響開發人員速度的幾個問題，但也有一些實踐可以解決規模問題。啟動產品的初始團隊可以少於10人，每個人輪流負責部署和生產監控。隨著時間的推移，你的團隊可能會發展到數百人，其中的子團隊負責特定的功能。隨著這種情況的發生和組織規模的擴大，每次部署中的更改數量和每次發佈嘗試中的風險量都呈超線性增長。每次發佈都包含數月的汗水和淚水。使發佈成功成為一項高度接觸和勞動密集型的工作。開發人員經常會被發現試圖決定哪一個更糟糕：放棄一個包含四分之一新特性和錯誤修復的版本，或者在對其品質沒有信心的情況下推出一個版本。

At scale, increased complexity usually manifests as increased release latency. Even if you release every day, a release can take a week or longer to fully roll out safely, leaving you a week behind when trying to debug any issues. This is where “Always Be Deploying” can return a development project to effective form. Frequent release trains allow for minimal divergence from a known good position, with the recency of changes aiding in resolving issues. But how can a team ensure that the complexity inherent with a large and quickly expanding codebase doesn’t weigh down progress?

在規模上，複雜性的增加通常表現為發佈延遲的增加。即使你每天都發布，一個版本也可能需要一週或更長的時間才能完全安全地發佈，當你試圖除錯任何問題時，就會落後一週。這就是 "始終部署 "可以使開發專案恢復到有效狀態的地方。頻繁的發版班車允許從一個已知的良好狀態中產生最小的偏差，變化的頻率有助於解決問題。但是，一個團隊如何才能確保一個龐大而快速擴充套件的程式碼函式庫所固有的複雜性不會拖累進度呢？

On Google Maps, we take the perspective that features are very important, but only very seldom is any feature so important that a release should be held for it. If releases are frequent, the pain a feature feels for missing a release is small in comparison to the pain all the new features in a release feel for a delay, and especially the pain users can feel if a not-quite-ready feature is rushed to be included.

在 Google 地圖上，我們的觀點是：功能是非常重要的，但只有在非常少的情況下，才會有如此重要的功能需要發佈。如果發佈的頻率很高，那麼一個功能因為錯過了一個版本而感到的痛苦與一個版本中所有的新功能因為延遲而感到的痛苦相比是很小的，特別是如果一個還沒有準備好的功能被匆忙納入，使用者會感到痛苦。

One release responsibility is to protect the product from the developers.

一個發佈責任是保護產品不受開發人員的影響。

When making trade-offs, the passion and urgency a developer feels about launching a new feature can never trump the user experience with an existing product. This means that new features must be isolated from other components via interfaces with strong contracts, separation of concerns, rigorous testing, communication early and often, and conventions for new feature acceptance.

在進行權衡時，開發人員對推出新功能的熱情和緊迫感永遠不能超過對現有產品的使用者體驗。這意味著，新功能必須透過具有強大契約的介面、關注點分離、嚴格測試、早期和經常的溝通以及新功能驗收的慣例，與其他元件隔離。

## Conclusion 總結

Over the years and across all of our software products, we’ve found that, counterintuitively, faster is safer. The health of your product and the speed of development are not actually in opposition to each other, and products that release more frequently and in small batches have better quality outcomes. They adapt faster to bugs encountered in the wild and to unexpected market shifts. Not only that, faster is *cheaper*, because having a predictable, frequent release train forces you to drive down the cost of each release and makes the cost of any abandoned release very low.

多年來，在我們所有的軟體產品中，我們發現，相反，更快更安全。你的產品的健康狀況和開發速度實際上並不是相互對立的，更頻繁和小批量發佈的產品具有更好的品質結果。它們能更快地適應在野外遇到的錯誤和意外的市場變化。不僅如此，更快就是*便宜*，因為有一個可預測的、頻繁的發版班車，迫使你降低每個版本的成本，並使任何被放棄的發佈的成本非常低。

Simply having the structures in place that *enable* continuous deployment generates the majority of the value, *even if you don’t actually push those releases out to users*. What do we mean? We don’t actually release a wildly different version of Search, Maps, or YouTube every day, but to be able to do so requires a robust, well- documented continuous deployment process, accurate and real-time metrics on user satisfaction and product health, and a coordinated team with clear policies on what makes it in or out and why. In practice, getting this right often also requires binaries that can be configured in production, configuration managed like code (in version control), and a toolchain that allows safety measures like dry-run verification, rollback/rollforward mechanisms, and reliable patching.

僅僅擁有*能夠*持續部署的結構，就能產生大部分的價值，*即使你沒有真正把這些版本推送給使用者*。我們的意思是什麼呢？我們實際上並不是每天都發佈一個完全不同的搜尋、地圖或YouTube的版本，但要做到這一點，就需要一個健壯的、有良好文件記錄的連續部署過程、關於使用者滿意度和產品健康狀況的準確即時指標，以及一個協調一致的團隊，該團隊擁有明確的策略，以確定成功與否以及原因。在實踐中，要做到這一點，往往還需要可以在生產中配置的二進位制套件，像程式碼一樣管理的配置（在版本控制中），以及一個可以採取安全措施的工具鏈，如干執行驗證、回滾/前滾機制和可靠的補丁。

## TL;DRs  內容提要

- *Velocity is a team sport*: The optimal workflow for a large team that develops code collaboratively requires modularity of architecture and near-continuous integration.

- Evaluate changes in isolation: Flag guard any features to be able to isolate prob‐ lems early.

- Make reality your benchmark: Use a staged rollout to address device diversity and the breadth of the userbase. Release qualification in a synthetic environment that isn’t similar to the production environment can lead to late surprises.

- Ship only what gets used: Monitor the cost and value of any feature in the wild to know whether it’s still relevant and delivering sufficient user value.

- Shift left: Enable faster, more data-driven decision making earlier on all changes through CI and continuous deployment.

- Faster is safer: Ship early and often and in small batches to reduce the risk of each release and to minimize time to market.

- *速度是一項團隊運動*。協作開發程式碼的大型團隊的最佳工作流程需要架構的模組化和近乎連續的整合。

- 孤立地評估變化。對任何功能進行標記，以便能夠儘早隔離問題。

- 讓現實成為你的基準。使用分階段推出的方式來解決裝置的多樣性和使用者群的廣泛性。在一個與生產環境不相似的合成環境中進行發佈鑑定，會導致後期的意外。

- 只發布被使用的東西。監控任何功能的成本和價值，以瞭解它是否仍有意義，是否能提供足夠的使用者價值。

- 向左移動。透過CI和持續部署，使所有的變化更快，更多的資料驅動的決策更早。

- 更快是更安全的。儘早地、經常地、小批量地發佈，以減少每次發佈的風險，並儘量縮短上市時間。





