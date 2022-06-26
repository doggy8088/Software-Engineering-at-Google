
**CHAPTER 22**

# Large-Scale Changes

# 第二十二章 大規模變更

                       							 Written by Hyrum Wright Edited by Lisa Carey

Think for a moment about your own codebase. How many files can you reliably update in a single, simultaneous commit? What are the factors that constrain that number? Have you ever tried committing a change that large? Would you be able to do it in a reasonable amount of time in an emergency? How does your largest commit size compare to the actual size of your codebase? How would you test such a change? How many people would need to review the change before it is committed? Would you be able to roll back that change if it did get committed? The answers to these questions might surprise you (both what you *think* the answers are and what they actually turn out to be for your organization).

考慮一下你自己的程式碼函式庫。在一次同步提交中，你可以可靠地更新多少個檔案？限制這一數字的因素有哪些？你有沒有試過做出這麼大的改變？在緊急情況下，你能在合理的時間內完成嗎？您的最大提交大小與程式碼函式庫的實際大小相比如何？你將如何測試這種變更？在提交更改之前，需要多少人進行審查？如果它確實被提交，你是否能夠回滾該更改？這些問題的答案可能會讓你大吃一驚（無論是你*認為*答案是什麼，還是它們對你的組織來說實際是什麼）。

At Google, we’ve long ago abandoned the idea of making sweeping changes across our codebase in these types of large atomic changes. Our observation has been that, as a codebase and the number of engineers working in it grows, the largest atomic change possible counterintuitively *decreases—*running all affected presubmit checks and tests becomes difficult, to say nothing of even ensuring that every file in the change is up to date before submission. As it has become more difficult to make sweeping changes to our codebase, given our general desire to be able to continually improve underlying infrastructure, we’ve had to develop new ways of reasoning about large-scale changes and how to implement them.

在谷歌，我們很久以前就放棄了在這些型別的大型原子性對程式碼函式庫進行徹底更改的想法。我們的觀察結果是，隨著程式碼函式庫和在其中工作的工程師數量的增加，最大的原子性更改可能會反直覺地減少執行所有受影響的提交前檢查和測試變得困難，更不用說確保更改中的每個檔案在提交前都是最新的了。隨著對程式碼函式庫進行全面更改變得越來越困難，考慮到我們希望能夠持續改進底層基礎設施的普遍願望，我們不得不開發新的方法來推理大規模更改以及如何實現這些更改。

In this chapter, we’ll talk about the techniques, both social and technical, that enable us to keep the large Google codebase flexible and responsive to changes in underlying infrastructure. We’ll also provide some real-life examples of how and where we’ve used these approaches. Although your codebase might not look like Google’s, understanding these principles and adapting them locally will help your development organization scale while still being able to make broad changes across your codebase.

在這一章中，我們將談論社會和技術方面的技術，這些技術使我們能夠保持大型谷歌程式碼函式庫的靈活性，並對底層基礎設施的變化做出響應。我們還將提供一些實際例子，說明我們如何以及在何處使用這些方法。儘管你的程式碼函式庫可能不像谷歌的程式碼函式庫，但瞭解這些原則並對其進行區域性調整，將有助於你的開發組織在擴大規模的同時，仍然能夠對你的程式碼函式庫進行廣泛的修改。

## What Is a Large-Scale Change? 什麼是大規模的變更？

Before going much further, we should dig into what qualifies as a large-scale change (LSC). In our experience, an LSC is any set of changes that are logically related but cannot practically be submitted as a single atomic unit. This might be because it touches so many files that the underlying tooling can’t commit them all at once, or it might be because the change is so large that it would always have merge conflicts. In many cases, an LSC is dictated by your repository topology: if your organization uses a collection of distributed or federated repositories,[1](#_bookmark1939) making atomic changes across them might not even be technically possible.[2](#_bookmark1940) We’ll look at potential barriers to atomic changes in more detail later in this chapter.

在進一步討論之前，我們應該探討一下什麼是大規模變更（LSC）。根據我們的經驗，LSC是指任何一組邏輯上相關但實際上不能作為一個單一的原子單元提交的變更。這可能是因為它涉及到檔案太多，以至於底層工具無法一次性提交所有檔案，也可能是因為變化太大，總是會有合併衝突。在很多情況下，LSC是由你的版本函式庫拓撲結構決定的：如果你的組織使用分散式或聯邦版本函式庫集合，在它們之間進行原子修改在技術上可能是不可能的。我們將在本章後面詳細討論原子變更的潛在障礙。

LSCs at Google are almost always generated using automated tooling. Reasons for making an LSC vary, but the changes themselves generally fall into a few basic categories:

•    Cleaning up common antipatterns using codebase-wide analysis tooling

•    Replacing uses of deprecated library features

•    Enabling low-level infrastructure improvements, such as compiler upgrades

•    Moving users from an old system to a newer one[3](#_bookmark1941)

谷歌的LSC幾乎都是使用自動工具產生的。製作LSC的原因各不相同，但修改本身通常分為幾個基本類別：

- 使用程式碼函式庫範圍內的分析工具來清理常見的反模式

- 替換已廢棄的函式庫特性的使用

- 實現底層基礎架構改進，如編譯器升級

- 將使用者從舊系統轉移到新系統

The number of engineers working on these specific tasks in a given organization might be low, but it is useful for their customers to have insight into the LSC tools and process. By their very nature, LSCs will affect a large number of customers, and the LSC tools easily scale down to teams making only a few dozen related changes.

在一個特定的組織中，從事這些特定任務的工程師的數量可能不多，但對於他們的客戶來說，深入瞭解LSC工具和流程是很有用的。就其性質而言，LSC將影響大量的客戶，而LSC工具很容易擴充套件到只做幾十個相關更改的團隊。

There can be broader motivating causes behind specific LSCs. For example, a new language standard might introduce a more efficient idiom for accomplishing a given task, an internal library interface might change, or a new compiler release might require fixing existing problems that would be flagged as errors by the new release. The majority of LSCs across Google actually have near-zero functional impact: they tend to be widespread textual updates for clarity, optimization, or future compatibility. But LSCs are not theoretically limited to this behavior-preserving/refactoring class of change.

在特定的LSC背後可能有更廣泛的動機。例如，新的語言標準可能會引入一種更有效的習慣用法來完成給定的任務，內部函式庫介面可能會更改，或者新的編譯器版本可能需要修復新版本標記為錯誤的現有問題。谷歌的大多數LSC實際上幾乎沒有功能影響：它們往往是為了清晰、優化或未來相容性而進行的廣泛文字更新。但從理論上講，LSC並不侷限於這種行為維護/重構類別的變化。

In all of these cases, on a codebase the size of Google’s, infrastructure teams might routinely need to change hundreds of thousands of individual references to the old pattern or symbol. In the largest cases so far, we’ve touched millions of references, and we expect the process to continue to scale well. Generally, we’ve found it advantageous to invest early and often in tooling to enable LSCs for the many teams doing infrastructure work. We’ve also found that efficient tooling also helps engineers performing smaller changes. The same tools that make changing thousands of files efficient also scale down to tens of files reasonably well.

在所有這些情況下，在像谷歌這樣規模的程式碼函式庫中，基礎設施團隊可能經常需要改變數十萬個對舊模式或符號的單獨參考。在迄今為止最大的案例中，我們已經觸及了數百萬個參考，而且我們希望這個過程能夠繼續良好地擴充套件。一般來說，我們發現儘早且經常投資於工具，以便為許多從事基礎設施工作的團隊啟用LSC是一種優勢。我們還發現，高效的工具也有助於工程師進行更小的更改。同樣的工具可以有效地更改數千個檔案，也可以很好地擴充套件到數十個檔案。

```
1  For some ideas about why, see [Chapter 16](#_bookmark1364).
2  It’s possible in this federated world to say “we’ll just commit to each repo as fast as possible to keep the duration of the build break small!” But that approach really doesn’t scale as the number of federated repositories grows.
3  For a further discussion about this practice, see [Chapter 15](#_bookmark1319).

1 關於原因的一些想法，見[第16章](#_bookmark1364)。
2 在這個聯合的世界裡，我們可以說 "我們將盡可能快地提交到每個 repo，以保持較小的建構中斷時間！" 但這種方法實際上不能隨著聯合儲存函式庫數量的增長而擴充套件。
3 關於這種做法的進一步討論，見第15章。
```

## Who Deals with LSCs? 誰負責處理LSC？

As just indicated, the infrastructure teams that build and manage our systems are responsible for much of the work of performing LSCs, but the tools and resources are available across the company. If you skipped [Chapter 1](#_bookmark3), you might wonder why infrastructure teams are the ones responsible for this work. Why can’t we just introduce a new class, function, or system and dictate that everybody who uses the old one move to the updated analogue? Although this might seem easier in practice, it turns out not to scale very well for several reasons.

如前所述，建構和管理我們系統的基礎架構團隊負責執行LSC的大部分工作，但工具和資源在整個公司都可用。如果你跳過了第1章，您可能會想，為什麼基礎設施團隊負責這項工作。為什麼我們不能引入一個新的類別、函式或系統，並要求所有使用舊類別、函式或系統的人都使用更新後的類別、函式或系統？雖然這在實踐中似乎更容易實現，但由於幾個原因，它的擴充套件性不是很好。

First, the infrastructure teams that build and manage the underlying systems are also the ones with the domain knowledge required to fix the hundreds of thousands of references to them. Teams that consume the infrastructure are unlikely to have the context for handling many of these migrations, and it is globally inefficient to expect them to each relearn expertise that infrastructure teams already have. Centralization also allows for faster recovery when faced with errors because errors generally fall into a small set of categories, and the team running the migration can have a playbook—formal or informal—for addressing them.

首先，建構和管理底層系統的基礎設施團隊也具備修復數十萬對它們的參考所需的領域知識。使用基礎架構的團隊不太可能具備處理許多此類別遷移的背景，並且期望他們重新學習基礎架構團隊已經具備的專業技能在全球範圍內是低效的。集中化處理還允許在遇到錯誤時更快地恢復，因為錯誤通常屬於一小部分類別，執行遷移的團隊可以有一個正式或非正式的預案來解決這些錯誤。

Consider the amount of time it takes to do the first of a series of semi-mechanical changes that you don’t understand. You probably spend some time reading about the motivation and nature of the change, find an easy example, try to follow the provided suggestions, and then try to apply that to your local code. Repeating this for every team in an organization greatly increases the overall cost of execution. By making only a few centralized teams responsible for LSCs, Google both internalizes those costs and drives them down by making it possible for the change to happen more efficiently.

考慮一下做一系列你不理解的半自動化變更中的第一次所需的時間。你可能會花一些時間來閱讀關於更改的動機和性質，找到一個簡單的例子，嘗試遵循所提供的建議，然後嘗試將其應用於你的原生代碼。對組織中的每個團隊重複此操作會大大增加執行的總體成本。透過只讓幾個集中的團隊負責LSC，谷歌將這些成本內部化，並透過使變革更有效地發生來降低成本。

Second, nobody likes unfunded mandates.[4](#_bookmark1944) Even though a new system might be categorically better than the one it replaces, those benefits are often diffused across an organization and thus unlikely to matter enough for individual teams to want to update on their own initiative. If the new system is important enough to migrate to, the costs of migration will be borne somewhere in the organization. Centralizing the migration and accounting for its costs is almost always faster and cheaper than depending on individual teams to organically migrate.

第二，沒有人喜歡沒有資金支援的任務。即使一個新的系統在本質上可能比它所取代的系統更好，這些好處往往分散在整個組織中，因此不太可能重要到讓個別團隊想要主動更新。如果新系統足夠重要，需要遷移到新系統，那麼遷移的成本將由組織的某個部門承擔。集中遷移和核算其成本，幾乎總是比依靠各個團隊的有機遷移更快、更便宜。

Additionally, having teams that own the systems requiring LSCs helps align incentives to ensure the change gets done. In our experience, organic migrations are unlikely to fully succeed, in part because engineers tend to use existing code as examples when writing new code. Having a team that has a vested interest in removing the old system responsible for the migration effort helps ensure that it actually gets done. Although funding and staffing a team to run these kinds of migrations can seem like an additional cost, it is actually just internalizing the externalities that an unfunded mandate creates, with the additional benefits of economies of scale.

此外，擁有需要LSC的系統的團隊有助於調整激勵機制，以確保完成更改。根據我們的經驗，有機遷移不太可能完全成功，部分原因是工程師在編寫新程式碼時傾向於使用現有程式碼作為例子。由一個對移除舊系統有既得利益的團隊負責遷移工作，有助於確保遷移工作真正完成。儘管為一個團隊提供資金和人員配置來執行這類別遷移似乎是一項額外的成本，但它實際上只是將沒有資金的授權所產生的外部性內部化，並帶來規模經濟的額外好處。

```
4  By “unfunded mandate,” we mean “additional requirements imposed by an external entity without balancing compensation.” Sort of like when the CEO says that everybody must wear an evening gown for “formal Fridays” but doesn’t give you a corresponding raise to pay for your formal wear.
4  我們所說的“無資金授權”是指“外部實體在不平衡薪酬的情況下強加的額外要求”。有點像CEO說每個人都必須在“正式週五”穿晚禮服，但沒有給你相應的加薪來支付正式著裝的費用。
```

-----

##### Case Study: Filling Potholes 案例研究：填補坑洞

Although the LSC systems at Google are used for high-priority migrations, we’ve also discovered that just having them available opens up opportunities for various small fixes across our codebase, which just wouldn’t have been possible without them. Much like transportation infrastructure tasks consist of building new roads as well as repairing old ones, infrastructure groups at Google spend a lot of time fixing existing code, in addition to developing new systems and moving users to them.

儘管谷歌的LSC系統用於高優先順序遷移，但我們也發現，只要有它們，就可以在我們的程式碼函式庫中提供各種小補丁，沒有它們是不可能的。就像交通基礎設施任務包括修建新道路和修復舊道路一樣，谷歌的基礎設施團隊除了開發新系統和將使用者轉移到新系統之外，還花費大量時間修復現有程式碼。

For example, early in our history, a template library emerged to supplement the C++ Standard Template Library. Aptly named the Google Template Library, this library consisted of several header files’ worth of implementation. For reasons lost in the mists of time, one of these header files was named *stl_util.h* and another was named *map-util.h* (note the different separators in the file names). In addition to driving the consistency purists nuts, this difference also led to reduced productivity, and engineers had to remember which file used which separator, and only discovered when they got it wrong after a potentially lengthy compile cycle.

例如，在我們歷史的早期，出現了一個範本函式庫來補充C++標準範本函式庫。這個函式庫被恰當地命名為谷歌範本函式庫，它包括幾個標頭檔案的實現。由於時間上的原因，其中一個頭檔案被命名為*stl_util.h*，另一個被命名為*map-util.h*（注意檔名中的不同分隔符）。除了讓純粹的一致性主義者發瘋之外，這種差異也導致了生產力的下降，工程師們不得不記住哪個檔案使用了哪個分隔符，只有在他們在潛在的漫長的編譯週期中弄錯了才會發現。

Although fixing this single-character change might seem pointless, particularly across a codebase the size of Google’s, the maturity of our LSC tooling and process enabled us to do it with just a couple weeks’ worth of background-task effort. Library authors could find and apply this change en masse without having to bother end users of these files, and we were able to quantitatively reduce the number of build failures caused by this specific issue. The resulting increases in productivity (and happiness) more than paid for the time to make the change.

雖然修復這個單一字元的變化看起來毫無意義，尤其是在像谷歌這樣規模的程式碼函式庫中，但我們的LSC工具和流程的成熟度使我們只需花幾周的時間就能完成這個任務。函式庫的作者可以發現並應用這一變化，而不必打擾這些檔案的終端使用者，我們能夠從數量上減少由這一特定問題引起的建構失敗的數量。由此帶來的生產力（和幸福感）的提高超過了做這個改變的時間成本。

As the ability to make changes across our entire codebase has improved, the diversity of changes has also expanded, and we can make some engineering decisions knowing that they aren’t immutable in the future. Sometimes, it’s worth the effort to fill a few potholes.

隨著在整個程式碼函式庫中進行更改的能力的提高，更改的多樣性也得到了擴充套件，我們可以做出一些工程決策，知道這些決策在未來並非一成不變。有時，為填補一些坑洞而付出努力是值得的。

-----

## Barriers to Atomic Changes  原子變更的障礙

Before we discuss the process that Google uses to actually effect LSCs, we should talk about why many kinds of changes can’t be committed atomically. In an ideal world, all logical changes could be packaged into a single atomic commit that could be tested, reviewed, and committed independent of other changes. Unfortunately, as a repository—and the number of engineers working in it—grows, that ideal becomes less feasible. It can be completely infeasible even at small scale when using a set of distributed or federated repositories.

在我們討論Google實際影響LSC的過程之前，我們應該先談談為什麼很多種類別的更改不能原子化地提交。在理想情況下，所有邏輯更改都可以打包成單個原子提交，可以獨立於其他更改進行測試、審查和提交。不幸的是，隨著版本函式庫和在其中工作的工程師數量的增加，這種理想變得不太可行。當使用一組分散式或聯邦版本函式庫時，即使在小規模下也完全不可行。

### Technical Limitations  技術限制

To begin with, most Version Control Systems (VCSs) have operations that scale linearly with the size of a change. Your system might be able to handle small commits (e.g., on the order of tens of files) just fine, but might not have sufficient memory or processing power to atomically commit thousands of files at once. In centralized VCSs, commits can block other writers (and in older systems, readers) from using the system as they process, meaning that large commits stall other users of the system.

首先，大多數版本控制系統（VCS）的操作都會隨著更改的大小進行線性擴充套件。你的系統可能能夠很好地處理小規模提交（例如，幾十個檔案的數量），但可能沒有足夠的記憶體或處理能力來一次性提交成千上萬的檔案。在集中式VCS中，提交會阻止其他寫入程式（以及在舊系統中的讀卡器）在處理時使用系統，這意味著大型提交會使系統的其他使用者陷入停滯。

In short, it might not be just “difficult” or “unwise” to make a large change atomically: it might simply be impossible with a given infrastructure. Splitting the large change into smaller, independent chunks gets around these limitations, although it makes the execution of the change more complex.[5](#_bookmark1953)

簡言之，以原子方式進行大規模更改可能不僅僅是“困難”或“不明智的”：對於給定的基礎設施，這可能根本不可能。將較大的更改拆分為較小的獨立塊可以繞過這些限制，儘管這會使更改的執行更加複雜。

```
5  See [*https://ieeexplore.ieee.org/abstract/document/8443579*](https://ieeexplore.ieee.org/abstract/document/8443579).
```
### Merge Conflicts 合併衝突

As the size of a change grows, the potential for merge conflicts also increases. Every version control system we know of requires updating and merging, potentially with manual resolution, if a newer version of a file exists in the central repository. As the number of files in a change increases, the probability of encountering a merge conflict also grows and is compounded by the number of engineers working in the repository.

隨著變更規模的增加，合併衝突的可能性也會增加。我們知道的每個版本控制系統都需要更新和合並，如果中央版本函式庫中存在較新版本的檔案，則可能需要手動解析。隨著更改中檔案數量的增加，遇到合併衝突的可能性也會增加，並且在版本函式庫中工作的工程師數量也會增加。

If your company is small, you might be able to sneak in a change that touches every file in the repository on a weekend when nobody is doing development. Or you might have an informal system of grabbing the global repository lock by passing a virtual (or even physical!) token around your development team. At a large, global company like Google, these approaches are just not feasible: somebody is always making changes to the repository.

如果你的公司很小，你可能會在週末沒有人做開發的時候，偷偷地修改版本函式庫中的每個檔案。或者你可能有一個非正式的系統，透過在開發團隊中傳遞一個虛擬的（甚至是物理的！）令牌來抓取全域性的版本函式庫鎖。在谷歌這樣的大公司，這些方法是不可行的：總有人在對版本函式庫進行修改。

With few files in a change, the probability of merge conflicts shrinks, so they are more likely to be committed without problems. This property also holds for the following areas as well.

由於更改中的檔案很少，合併衝突的可能性會減小，因此它們更有可能在提交時不會出現問題。該屬性也適用於以下區域。

### No Haunted Graveyards  沒有鬧鬼的墓地

The SREs who run Google’s production services have a mantra: “No Haunted Graveyards.” A haunted graveyard in this sense is a system that is so ancient, obtuse, or complex that no one dares enter it. Haunted graveyards are often business-critical systems that are frozen in time because any attempt to change them could cause the system to fail in incomprehensible ways, costing the business real money. They pose a real existential risk and can consume an inordinate amount of resources.

運營谷歌生產服務的SRE們有一句格言：“沒有鬧鬼墓地”。從這個意義上說，鬧鬼墓地是一個如此古老、遲鈍或複雜的系統，以至於沒有人敢進入它。鬧鬼的墓地往往是被凍結的關鍵業務系統，因為任何試圖改變它們的行為都可能導致系統以無法理解的方式失敗，從而使企業付出實實在在的代價。它們構成了真正的生存風險，並可能消耗過多的資源。

Haunted graveyards don’t just exist in production systems, however; they can be found in codebases. Many organizations have bits of software that are old and unmaintained, written by someone long off the team, and on the critical path of some important revenue-generating functionality. These systems are also frozen in time, with layers of bureaucracy built up to prevent changes that might cause instability. Nobody wants to be the network support engineer II who flipped the wrong bit!

然而，鬧鬼的墓地並不僅僅存在於生產系統中，它們也可以在程式碼函式庫中找到。許多組織都有一些老舊的、未經維護的軟體，它們是由早已離開團隊的人編寫的，並且處於一些重要的創收功能的關鍵路徑上。這些系統也被凍結在時間中，層層疊疊的官僚機建構立起來，防止可能導致不穩定的變化。沒有人想成為網路支援工程師，他犯了錯誤！

These parts of a codebase are anathema to the LSC process because they prevent the completion of large migrations, the decommissioning of other systems upon which they rely, or the upgrade of compilers or libraries that they use. From an LSC perspective, haunted graveyards prevent all kinds of meaningful progress.

程式碼函式庫的這些部分是LSC過程的詛咒，因為它們阻止了大型遷移的完成、它們所依賴的其他系統的退役，或者它們所使用的編譯器或函式庫的升級。從LSC的角度來看，鬧鬼的墓地阻止了各種有意義的進步。

At Google, we’ve found the counter to this to be good, old-fashioned  testing. When software is thoroughly tested, we can make arbitrary changes to it and know with confidence whether those changes are breaking, no matter the age or complexity of the system. Writing those tests takes a lot of effort, but it allows a codebase like Google’s to evolve over long periods of time, consigning the notion of haunted software graveyards to a graveyard of its own.

在谷歌，我們發現這是一個好的、老式的測試。當軟體經過徹底測試後，我們可以對其進行任意更改，並有信心地知道這些更改是否正在中斷，無論系統的時間或複雜性如何。編寫這些測試需要很多努力，但它允許像谷歌這樣的程式碼函式庫在很長一段時間內進化，將鬧鬼軟體墓地的概念交付給它自己的墓地。

### Heterogeneity  異質性

LSCs really work only when the bulk of the effort for them can be done by computers, not humans. As good as humans can be with ambiguity, computers rely upon consistent environments to apply the proper code transformations to the correct places. If your organization has many different VCSs, Continuous Integration (CI) systems, project-specific tooling, or formatting guidelines, it is difficult to make sweeping changes across your entire codebase. Simplifying the environment to add more consistency will help both the humans who need to move around in it and the robots making automated transformations.

只有當大部分的工作由計算機而不是人類來完成時，LSC才能真正發揮作用。儘管人類可以很好地處理模稜兩可的問題，但計算機依賴於一致的環境將正確的程式碼轉換應用到正確的位置。如果你的組織有許多不同的VCS、持續整合（CI）系統、特定專案的工具或格式化準則，就很難在整個程式碼函式庫中進行全面的更改。簡化環境以增加一致性將有助於需要在其中移動的人類和進行自動轉換的機器人。

For example, many projects at Google have presubmit tests configured to run before changes are made to their codebase. Those checks can be very complex, ranging from checking new dependencies against a whitelist, to running tests, to ensuring that the change has an associated bug. Many of these checks are relevant for teams writing new features, but for LSCs, they just add additional irrelevant complexity.

例如，谷歌的許多專案都配置了預提交測試，以便在對其程式碼函式庫進行修改之前執行。這些檢查可能非常複雜，從對照白名單檢查新的依賴關係，到執行測試，再到確保變化有相關的bug。這些檢查中有許多與編寫新功能的團隊有關，但對於LSC來說，它們只是增加了額外的無關的複雜性。

We’ve decided to embrace some of this complexity, such as running presubmit tests, by making it standard across our codebase. For other inconsistencies, we advise teams to omit their special checks when parts of LSCs touch their project code. Most teams are happy to help given the benefit these kinds of changes are to their projects.

我們已經決定採用這種複雜性，例如透過使其成為我們程式碼函式庫中的標準來執行預提交測試。對於其他不一致性，我們建議團隊在LSC的某些部分接觸到其專案程式碼時忽略其特殊檢查。鑑於此類別變更對其專案的好處，大多數團隊都樂於提供幫助。

### Testing  測試

Every change should be tested (a process we’ll talk about more in just a moment), but the larger the change, the more difficult it is to actually test it appropriately. Google’s CI system will run not only the tests immediately impacted by a change, but also any tests that transitively depend on the changed files.[6](#_bookmark1961) This means a change gets broad coverage, but we’ve also observed that the farther away in the dependency graph a test is from the impacted files, the more unlikely a failure is to have been caused by the change itself.

每個變更都應該進行測試（稍後我們將詳細討論這個過程），但是變更越大，實際測試它就越困難。Google的CI系統不僅會執行立即受到更改影響的測試，還會執行過渡依賴於更改檔案的任何測試。這意味著更改會得到廣泛的覆蓋，但我們還觀察到，在依賴關係圖中，測試距離受影響檔案越遠，失敗越不可能是由變化本身造成的。

Small, independent changes are easier to validate, because each of them affects a smaller set of tests, but also because test failures are easier to diagnose and fix. Finding the root cause of a test failure in a change of 25 files is pretty straightforward; finding 1 in a 10,000-file change is like the proverbial needle in a haystack.

小的、獨立的更改更容易驗證，因為每個更改都會影響較小的測試集，但也因為測試失敗更容易診斷和修復。在25個檔案的更改中找到測試失敗的根本原因非常簡單；在10,000個檔案更改中找到1個，就像諺語中大海撈針一樣。

The trade-off in this decision is that smaller changes will cause the same tests to be run multiple times, particularly tests that depend on large parts of the codebase. Because engineer time spent tracking down test failures is much more expensive than the compute time required to run these extra tests, we’ve made the conscious decision that this is a trade-off we’re willing to make. That same trade-off might not hold for all organizations, but it is worth examining what the proper balance is for yours.

這個決定的權衡是，較小的更改將導致相同的測試執行多次，特別是依賴於大部分程式碼函式庫的測試。因為工程師追蹤測試失敗所花費的時間比執行這些額外測試所需的計算時間要昂貴得多，所以我們有意識地決定，這是我們願意做出的權衡。這種權衡可能並不適用於所有組織，但值得研究的是，對於你的組織來說，什麼才是適當的平衡。

```
6  This probably sounds like overkill, and it likely is. We’re doing active research on the best way to determine the “right” set of tests for a given change, balancing the cost of compute time to run the tests, and the human cost of making the wrong choice.

6 這聽起來可能是矯枉過正，而且很可能是。我們正在積極研究為一個特定的變化確定 "正確 "的測試集的最佳方法，平衡執行測試的計算時間成本和做出錯誤選擇的人力成本。
```

-----

##### Case Study: Testing LSCs  案例研究：測試LSC

***Adam Bender***

Today it is common for a double-digit percentage (10% to 20%) of the changes in a project to be the result of LSCs, meaning a substantial amount of code is changed in projects by people whose full-time job is unrelated to those projects. Without good tests, such work would be impossible, and Google’s codebase would quickly atrophy under its own weight. LSCs enable us to systematically migrate our entire codebase to newer APIs, deprecate older APIs, change language versions, and remove popular but dangerous practices.

如今，一個專案中兩位數百分比（10%到20%）的變更是LSC的結果是很常見的，這意味著大量的程式碼是由全職工作與這些專案無關的人在專案中變更的。如果沒有良好的測試，這樣的工作將是不可能的，谷歌的程式碼函式庫將在自身的壓力下迅速萎縮。LSC使我們能夠系統地將整個程式碼函式庫遷移到較新的API，棄用較舊的API，更改語言版本，並刪除流行但危險的做法。

Even a simple one-line signature change becomes complicated when made in a thousand different places across hundreds of different products and services.[7](#_bookmark1965) After the change is written, you need to coordinate code reviews across dozens of teams. Lastly, after reviews are approved, you need to run as many tests as you can to be sure the change is safe.[8](#_bookmark1966) We say “as many as you can,” because a good-sized LSC could trigger a rerun of every single test at Google, and that can take a while. In fact, many LSCs have to plan time to catch downstream clients whose code backslides while the LSC makes its way through the process.

即使是一個簡單的單行簽名修改，如果在上百個不同的產品和服務的一千多個不同的地方進行，也會變得很複雜。修改寫完後，你需要協調幾十個團隊的程式碼審查。最後，在審查通過後，你需要執行儘可能多的測試，以確保變化是安全的。我們說 "儘可能多"，是因為一個規模不錯的LSC可能會觸發谷歌的每一個測試的重新執行，而這可能需要一段時間。事實上，許多LSC必須計劃好時間，以便在LSC進行的過程中抓住那些程式碼倒退的下游客戶。

Testing an LSC can be a slow and frustrating process. When a change is sufficiently large, your local environment is almost guaranteed to be permanently out of sync with head as the codebase shifts like sand around your work. In such circumstances, it is easy to find yourself running and rerunning tests just to ensure your changes continue to be valid. When a project has flaky tests or is missing unit test coverage, it can require a lot of manual intervention and slow down the entire process. To help speed things up, we use a strategy called the TAP (Test Automation Platform) train.

測試LSC可能是一個緩慢而令人沮喪的過程。當一個變更足夠大的時候，你的本地環境幾乎可以肯定會與head永久不同步，因為程式碼函式庫會像沙子一樣在你的工作中移動。在這種情況下，很容易發現自己在執行和重新執行測試，以確保你的變化繼續有效。當一個專案有不穩定的測試或缺少單元測試覆蓋率時，它可能需要大量的人工干預並拖慢整個過程。為了幫助加快進度，我們使用了一種叫做TAP（測試自動化平臺）的策略。

**Riding the TAP Train**  **搭乘TAP列車** 

The core insight to LSCs is that they rarely interact with one another, and most affected tests are going to pass for most LSCs. As a result, we can test more than one change at a time and reduce the total number of tests executed. The train model has proven to be very effective for testing LSCs.

對LSC的核心見解是，它們很少相互影響，對於大多數LSC來說，大多數受影響的測試都會透過。因此，我們可以一次測試一個以上的變化，減少執行的測試總數。事實證明，訓練模型對測試LSC非常有效。

The TAP train takes advantage of two facts:

- LSCs tend to be pure refactorings and therefore very narrow in scope, preserving local semantics.

- Individual changes are often simpler and highly scrutinized, so they are correct  more often than not.

TAP列車利用了兩個事實：
- LSC往往是純粹的重構，因此範圍非常窄，保留了本地語義。
- 單獨的修改通常比較簡單，而且受到高度審查，所以它們往往是正確的。

The train model also has the advantage that it works for multiple changes at the same time and doesn’t require that each individual change ride in isolation.[9](#_bookmark1972)

列車模型還有一個優點，即它同時適用於多個變化，不要求每個單獨的變化都是孤立的。

The train has five steps and is started fresh every three hours:

1.  For each change on the train, run a sample of 1,000 randomly-selected tests.
2.  Gather up all the changes that passed their 1,000 tests and create one uberchange from all of them: “the train.”
3.  Run the union of all tests directly affected by the group of changes. Given a large enough (or low-level enough) LSC, this can mean running every single test in Google’s repository. This process can take more than six hours to complete.
4.  For each nonflaky test that fails, rerun it individually against each change that made it into the train to determine which changes caused it to fail.
5.  TAP generates a report for each change that boarded the train. The report describes all passing and failing targets and can be used as evidence that an LSC is safe to submit.

列車模式有五個階段，每三小時重新啟動一次：

1.  對於列車上的每個變化，執行1000個隨機選擇的測試樣本。
2.  收集所有透過1000次測試的變化，並從所有這些變化中建立一個超級變化：”車次"。
3.  執行所有直接受該組變化影響的測試的聯合。如果LSC足夠大（或足夠底層），這可能意味著執行谷歌資源函式庫中的每一個測試。這個過程可能需要六個多小時來完成。
4.  對於每一個失敗的非漏洞測試，針對每一個進入火車的變化單獨重新執行它，以確定哪些變化導致它失敗。
5.  TAP為每個上火車的變化產生一份報告。該報告描述了所有透過和未透過的目標，可以作為LSC可以安全提交的證據。

-----

### Code Review 程式碼審查

Finally, as we mentioned in [Chapter 9](#_bookmark664), all changes need to be reviewed before submission, and this policy applies even for LSCs. Reviewing large commits can be tedious, onerous, and even error prone, particularly if the changes are generated by hand (a process you want to avoid, as we’ll discuss shortly). In just a moment, we’ll look at how tooling can often help in this space, but for some classes of changes, we still want humans to explicitly verify they are correct. Breaking an LSC into separate shards makes this much easier.

最後，正如我們在第9章中提到的，所有的修改都需要在提交前進行稽核，這個策略甚至適用於LSC。審閱大型提交可能會很乏味、繁重，甚至容易出錯，特別是如果這些修改是手工產生的（我們很快就會討論，這是一個你想避免的過程）。稍後，我們將看看工具化如何在這個領域提供幫助，但對於某些類別的修改，我們仍然希望人類明確地驗證它們是否正確。將一個LSC分解成獨立的片段，使之更容易。

-----

##### Case Study: scoped_ptr to std::unique_ptr  案例研究：scoped_ptr到std::unique_ptr

Since its earliest days, Google’s C++ codebase has had a self-destructing smart pointer for wrapping heap-allocated C++ objects and ensuring that they are destroyed when the smart pointer goes out of scope. This type was called scoped_ptr and was used extensively throughout Google’s codebase to ensure that object lifetimes were appropriately managed. It wasn’t perfect, but given the limitations of the then-current C++ standard (C++98) when the type was first introduced, it made for safer programs.

從最早期開始，Google的C++程式碼函式庫就有一個自毀的智慧指標，用於包裝堆分配的C++物件，並確保在智慧指標超出範圍時將其銷燬。這種型別被稱為scoped_ptr，在Google的程式碼函式庫中被廣泛使用，以確保物件的壽命得到適當的管理。它並不完美，但考慮到該型別首次引入時當時的C++標準（C++98）的限制，它使程式更加安全。

In C++11, the language introduced a new type: std::unique_ptr. It fulfilled the same function as scoped_ptr, but also prevented other classes of bugs that the language now could detect. std::unique_ptr was strictly better than scoped_ptr, yet Google’s codebase had more than 500,000 references to scoped_ptr scattered among millions of source files. Moving to the more modern type required the largest LSC attempted to that point within Google.

在C++11中，該語言引入了一個新的型別：std::unique_ptr。std::unique_ptr嚴格來說比scoped_ptr好，但Google的程式碼函式庫中有超過50萬個對scoped_ptr的參考，散佈在數百萬個原始檔中。向更現代的模式發展需要谷歌內部最大的LSC。

Over the course of several months, several engineers attacked the problem in parallel. Using Google’s large-scale migration infrastructure, we were able to change references to scoped_ptr into references to std::unique_ptr as well as slowly adapt scoped_ptr to behave more closely to std::unique_ptr. At the height of the migration process, we were consistently generating, testing and committing more than 700 independent changes, touching more than 15,000 files *per day*. Today, we sometimes manage 10 times that throughput, having refined our practices and improved our tooling.

在幾個月的時間裡，幾位工程師同時攻克了這個問題。利用谷歌的大規模遷移基礎設施，我們能夠將對scoped_ptr的參考改為對std::unique_ptr的參考，並慢慢調整scoped_ptr，使其行為更接近於std::unique_ptr。在遷移過程的高峰期，我們一直在產生、測試和提交超過700個獨立的變化，每天*觸及*超過15000個檔案。今天，在完善了我們的實踐和改進了我們的工具後，我們有時能管理10倍的吞吐量。

Like almost all LSCs, this one had a very long tail of tracking down various nuanced behavior dependencies (another manifestation of Hyrum’s Law), fighting race conditions with other engineers, and uses in generated code that weren’t detectable by our automated tooling. We continued to work on these manually as they were discovered by the testing infrastructure.

像幾乎所有的LSC一樣，這個LSC有一個長尾效應，那就是追蹤各種細微的行為依賴（Hyrum定律的另一種表現），與其他工程師一起對抗競賽條件，以及使用產生的程式碼，而我們的自動化工具是無法檢測到的。我們繼續手動處理這些問題，因為它們是由測試基礎設施發現的。

scoped_ptr was also used as a parameter type in some widely used APIs, which made small independent changes difficult. We contemplated writing a call-graph analysis system that could change an API and its callers, transitively, in one commit, but were concerned that the resulting changes would themselves be too large to commit atomically.

scoped_ptr在一些廣泛使用的API中也被用作引數型別，這使得小的獨立變化變得困難。我們考慮過編寫一個呼叫圖分析系統，它可以在一次提交中改變API及其呼叫者，但我們擔心由此產生的改變本身太大，無法原子提交。

In the end, we were able to finally remove scoped_ptr by first making it a type alias of std::unique_ptr and then performing the textual substitution between the old alias and the new, before eventually just removing the old scoped_ptr alias. Today, Google’s codebase benefits from using the same standard type as the rest of the C++ ecosystem, which was possible only because of our technology and tooling for LSCs.

最後，我們能夠最終刪除scoped_ptr，首先讓它成為std::unique_ptr的型別別名，然後在舊的別名和新的別名之間進行文字替換，最後只是刪除舊的scoped_ptr別名。今天，谷歌的程式碼函式庫從使用與C++生態系統其他部分相同的標準型別中受益，這可能是因為我們的技術和工具為LSC。

-----

## LSC Infrastructure  LSC基礎設施

Google has invested in a significant amount of infrastructure to make LSCs possible. This infrastructure includes tooling for change creation, change management, change review, and testing. However, perhaps the most important support for LSCs has been the evolution of cultural norms around large-scale changes and the oversight given to them. Although the sets of technical and social tools might differ for your organization, the general principles should be the same.

谷歌已經投資了大量的基礎設施，使LSC成為可能。這種基礎設施包括用於建立變更、變更管理、變更審查和測試的工具。然而，對LSC最重要的支援可能是圍繞大規模變化和對它們的監督的文化規範的演變。雖然你的組織的技術和社會工具集可能有所不同，但一般原則應該是相同的。

### Policies and Culture  策略和文化

As we’ve described in [Chapter 16](#_bookmark1364), Google stores the bulk of its source code in a single monolithic repository (monorepo), and every engineer has visibility into almost all of this code. This high degree of openness means that any engineer can edit any file and send those edits for review to those who can approve them. However, each of those edits has costs, both to generate as well as review.[10](#_bookmark1981)

正如我們在第16章中所描述的那樣，谷歌將其大部分原始碼儲存在單個程式碼函式庫（monorepo）中，每個工程師都可以看到幾乎所有這些程式碼。這種高度的開放性意味著任何工程師都可以編輯任何檔案，並將這些編輯傳送給可以批准它們的人進行審查。然而，每一個編輯都有成本，包括產生和審查。

Historically, these costs have been somewhat symmetric, which limited the scope of changes a single engineer or team could generate. As Google’s LSC tooling improved, it became easier to generate a large number of changes very cheaply, and it became equally easy for a single engineer to impose a burden on a large number of reviewers across the company. Even though we want to encourage widespread improvements to our codebase, we want to make sure there is some oversight and thoughtfulness behind them, rather than indiscriminate tweaking.[11](#_bookmark1982)

從歷史上看，這些成本在某種程度上是對稱的，這限制了單個工程師或團隊可能產生的變更範圍。隨著谷歌LSC工具的改進，以極低的成本產生大量更改變得更加容易，而對於單個工程師來說，給公司內的大量審閱者施加負擔也變得同樣容易。儘管我們希望鼓勵對我們的程式碼函式庫進行廣泛的改進，但我們希望確保在這些改進背後有一些疏忽和深思熟慮，而不是隨意的調整。

The end result is a lightweight approval process for teams and individuals seeking to make LSCs across Google. This process is overseen by a group of experienced engineers who are familiar with the nuances of various languages, as well as invited domain experts for the particular change in question. The goal of this process is not to prohibit LSCs, but to help change authors produce the best possible changes, which make the most use of Google’s technical and human capital. Occasionally, this group might suggest that a cleanup just isn’t worth it: for example, cleaning up a common typo without any way of preventing recurrence.

最終的結果是為尋求在谷歌範圍內進行LSC的團隊和個人提供了一個輕量級的審批過程。這個過程由一群經驗豐富的工程師監督，他們熟悉各種語言的細微差別，並邀請了相關特定變化的領域專家。這個過程的目的不是要禁止LSC，而是要幫助修改者產生儘可能好的修改，從而最大限度地利用谷歌的技術和人力資本。偶爾，這個小組可能會建議清理工作不值得做：例如，清理一個常見的錯別字，但沒有任何辦法防止再次發生。

Related to these policies was a shift in cultural norms surrounding LSCs. Although it is important for code owners to have a sense of responsibility for their software, they also needed to learn that LSCs were an important part of Google’s effort to scale our software engineering practices. Just as product teams are the most familiar with their own software, library infrastructure teams know the nuances of the infrastructure, and getting product teams to trust that domain expertise is an important step toward social acceptance of LSCs. As a result of this culture shift, local product teams have grown to trust LSC authors to make changes relevant to those authors’ domains.

與這些策略相關的是圍繞LSC的文化規範的轉變。雖然程式碼所有者對自己的軟體有責任感很重要，但他們也需要了解LSC是Google努力擴充套件軟體工程實踐的重要組成部分。正如產品團隊最熟悉自己的軟體一樣，基礎類別函式庫團隊也知道基礎設施的細微差別，讓產品團隊相信領域專業知識是LSC獲得社會認可的重要一步。作為這種文化轉變的結果，本地產品團隊已經開始信任LSC作者，讓他們做出與這些作者的領域相關的更改。

Occasionally, local owners question the purpose of a specific commit being made as part of a broader LSC, and change authors respond to these comments just as they would other review comments. Socially, it’s important that code owners understand the changes happening to their software, but they also have come to realize that they don’t hold a veto over the broader LSC. Over time, we’ve found that a good FAQ and a solid historic track record of improvements have generated widespread endorsement of LSCs throughout Google.

偶爾，本地所有者會質疑作為更廣泛的LSC的一部分的特定提交的目的，而變更作者會像迴應其他審查意見一樣迴應這些意見。從社會角度來說，程式碼所有者瞭解發生在他們軟體上的變化是很重要的，但他們也意識到他們對更廣泛的LSC並不擁有否決權。隨著時間的推移，我們發現，一個好的FAQ和一個可靠的歷史改進記錄已經在整個谷歌產生了對LSC的廣泛認可。

```
10  There are obvious technical costs here in terms of compute and storage, but the human costs in time to review a change far outweigh the technical ones.

11   For example, we do not want the resulting tools to be used as a mechanism to fight over the proper spelling of “gray” or “grey” in comments.

10  在計算和儲存方面存在明顯的技術成本，但及時審查變更所需的人力成本遠遠超過技術成本。
11  例如，我們不希望由此產生的工具被用作一種機制來爭奪評論中“灰色”或“灰色”的正確拼寫。
```

### Codebase Insight  程式碼函式庫的洞察力

To do LSCs, we’ve found it invaluable to be able to do large-scale analysis of our codebase, both on a textual level using traditional tools, as well as on a semantic level. For example, Google’s use of the semantic indexing tool [Kythe ](https://kythe.io/)provides a complete map of the links between parts of our codebase, allowing us to ask questions such as “Where are the callers of this function?” or “Which classes derive from this one?” Kythe and similar tools also provide programmatic access to their data so that they can be incorporated into refactoring tools. (For further examples, see Chapters [17 ](#_bookmark1485)and [20](#_bookmark1781).)

要進行LSC，我們發現能夠使用傳統工具在文字級別和語義級別上對程式碼函式庫進行大規模分析是非常寶貴的。例如，Google使用語義索引工具Kythe提供了程式碼函式庫各部分之間連結的完整地圖，允許我們提出諸如“此函式的呼叫方在哪裡？”或“哪些類別源自此函式？”Kythe和類似的工具還提供對其資料的程式設計訪問，以便可以將它們合併到重構工具中。（更多示例請參見第17章和第20章。）

We also use compiler-based indices to run abstract syntax tree-based analysis and transformations over our codebase. Tools such as [ClangMR](https://oreil.ly/c6xvO), JavacFlume, or [Refaster](https://oreil.ly/Er03J), which can perform transformations in a highly parallelizable way, depend on these insights as part of their function. For smaller changes, authors can use specialized, custom tools, perl or sed, regular expression matching, or even a simple shell script.

我們還使用基於編譯器的索引，在我們的程式碼函式庫上執行基於抽象語法樹的分析和轉換。諸如[ClangMR](https://oreil.ly/c6xvO)、JavacFlume或[Refaster](https://oreil.ly/Er03J)等工具，可以以高度可並行的方式進行轉換，其功能的一部分依賴於這些洞察力。對於較小的變化，作者可以使用專門的、訂製的工具、perl或sed、正則表示式匹配，甚至是一個簡單的shell指令碼。

Whatever tool your organization uses for change creation, it’s important that its human effort scale sublinearly with the codebase; in other words, it should take roughly the same amount of human time to generate the collection of all required changes, no matter the size of the repository. The change creation tooling should also be comprehensive across the codebase, so that an author can be assured that their change covers all of the cases they’re trying to fix.

無論你的組織使用什麼工具來建立變更，重要的是它的人力與程式碼函式庫成次線性擴充套件；換句話說，無論程式碼函式庫的大小，它都應該花費大致相同的人力時間來產生所有需要的變更集合。變更建立工具也應該在整個程式碼函式庫中是全面的，這樣作者就可以確信他們的變更涵蓋了他們試圖修復的所有情況。

As with other areas in this book, an early investment in tooling usually pays off in the short to medium term. As a rule of thumb, we’ve long held that if a change requires more than 500 edits, it’s usually more efficient for an engineer to learn and execute our change-generation tools rather than manually execute that edit. For experienced “code janitors,” that number is often much smaller.

與本書中的其他領域一樣，對工具的早期投資通常在中短期內獲得回報。根據經驗，我們一直認為，如果一個變更需要500次以上的編輯，工程師學習和執行我們的變更產生工具通常比手動執行該編輯更有效。對於有經驗的“程式碼管理員”，這個數字通常要小得多。

### Change Management  變更管理

Arguably the most important piece of large-scale change infrastructure is the set of tooling that shards a master change into smaller pieces and manages the process of testing, mailing, reviewing, and committing them independently. At Google, this tool is called Rosie, and we discuss its use more completely in a few moments when we examine our LSC process. In many respects, Rosie is not just a tool, but an entire platform for making LSCs at Google scale. It provides the ability to split the large sets of comprehensive changes produced by tooling into smaller shards, which can be tested, reviewed, and submitted independently.

可以說，大規模變更基礎設施中最重要的部分是一套工具，它將主變更分割成小塊，並獨立管理測試、推送、審查和提交的過程。在谷歌，這個工具被稱為Rosie，我們將在稍後檢查我們的LSC過程時更全面地討論它的使用。在許多方面，Rosie不僅僅是一個工具，而是一個在谷歌規模上製作LSC的整個平臺。它提供了一種能力，可以將工具產生的大型綜合修改集分割成較小的分片，這些分片可以被獨立測試、審查和提交。

### Testing  測試

Testing is another important piece of large-scale-change–enabling infrastructure. As discussed in [Chapter 11](#_bookmark838), tests are one of the important ways that we validate our software will behave as expected. This is particularly important when applying changes that are not authored by humans. A robust testing culture and infrastructure means that other tooling can be confident that these changes don’t have unintended effects.

測試是支援大規模變革的基礎設施的另一個重要部分。正如在第11章中所討論的，測試是我們驗證我們的軟體將按照預期行為的重要方法之一。這在應用非人工編寫的更改時尤為重要。一個強大的測試文化和基礎設施意味著其他工具可以確信這些更改不會產生意外的影響。

Google’s testing strategy for LSCs differs slightly from that of normal changes while still using the same underlying CI infrastructure. Testing LSCs means not just ensuring the large master change doesn’t cause failures, but that each shard can be submitted safely and independently. Because each shard can contain arbitrary files, we don’t use the standard project-based presubmit tests. Instead, we run each shard over the transitive closure of every test it might affect, which we discussed earlier.

谷歌針對LSC的測試策略與普通更改略有不同，但仍使用相同的底層CI基礎設施。測試LSC不僅意味著確保大型主更改不會導致失敗，而且還意味著可以安全、獨立地提交每個分片。因為每個分片可以包含任意檔案，所以我們不使用標準的基於專案的預提交測試。相反，我們在它可能影響的每個測試的可傳遞閉包上執行每個分片，我們在前面討論過。

### Language Support  程式語言支援

LSCs at Google are typically done on a per-language basis, and some languages support them much more easily than others. We’ve found that language features such as type aliasing and forwarding functions are invaluable for allowing existing users to continue to function while we introduce new systems and migrate users to them nonatomically. For languages that lack these features, it is often difficult to migrate systems incrementally.[12](#_bookmark1993)

谷歌的LSC通常以每種程式語言為基礎，有些語言比其他語言更容易支援LSC。我們發現，在我們引入新系統並以非原子方式將使用者遷移到這些系統時，諸如型別別名和轉發功能之類別的語言功能對於允許現有使用者繼續工作是非常寶貴的。對於缺少這些功能的程式語言，通常很難增量遷移系統。

We’ve also found that statically typed languages are much easier to perform large automated changes in than dynamically typed languages. Compiler-based tools along with strong static analysis provide a significant amount of information that we can use to build tools to affect LSCs and reject invalid transformations before they even get to the testing phase. The unfortunate result of this is that languages like Python, Ruby, and JavaScript that are dynamically typed are extra difficult for maintainers. Language choice is, in many respects, intimately tied to the question of code lifespan: languages that tend to be viewed as more focused on developer productivity tend to be more difficult to maintain. Although this isn’t an intrinsic design requirement, it is where the current state of the art happens to be.

我們還發現，靜態型別的語言比動態型別的語言更容易進行大規模的自動化修改。基於編譯器的工具以及強大的靜態分析提供了大量的資訊，我們可以利用這些資訊來建立影響LSC的工具，並在它們進入測試階段之前拒絕無效的轉換。這樣做的不幸結果是，像Python、Ruby和JavaScript這些動態型別的語言對維護者來說是額外困難的。在許多方面，程式語言的選擇與程式碼壽命的問題密切相關：那些傾向於被視為更注重開發者生產力的程式語言往往更難維護。雖然這不是一個固有的設計要求，但這是目前的技術狀況。

Finally, it’s worth pointing out that automatic language formatters are a crucial part of the LSC infrastructure. Because we work toward optimizing our code for readability, we want to make sure that any changes produced by automated tooling are intelligible to both immediate reviewers and future readers of the code. All of the LSCgeneration tools run the automated formatter appropriate to the language being changed as a separate pass so that the change-specific tooling does not need to concern itself with formatting specifics. Applying automated formatting, such as [google-java-format ](https://github.com/google/google-java-format)or [clang-format](https://clang.llvm.org/docs/ClangFormat.html), to our codebase means that automatically produced changes will “fit in” with code written by a human, reducing future development friction. Without automated formatting, large-scale automated changes would never have become the accepted status quo at Google.

最後，值得指出的是，自動語言格式化程式是LSC基礎設施的一個重要組成部分。因為我們致力於優化我們的程式碼的可讀性，我們希望確保任何由自動工具產生的變化對即時的審查者和未來的程式碼讀者來說都是可理解的。所有的LSC產生工具都將適合於被修改的語言的自動格式化器作為一個單獨的通道來執行，這樣，針對修改的工具就不需要關注格式化的細節了。將自動格式化，如[google-java-format](https://github.com/google/google-java-format)或[clang-format](https://clang.llvm.org/docs/ClangFormat.html)，應用到我們的程式碼函式庫中，意味著自動產生的變化將與人類編寫的程式碼 “合併"，減少未來的開發阻力。如果沒有自動格式化，大規模的自動修改就永遠不會成為谷歌的公認現狀。

```
12   In fact, Go recently introduced these kinds of language features specifically to support large-scale refactorings （ see [https://talks.golang.org/2016/refactor.article](https://talks.golang.org/2016/refactor.article) ）.
12  事實上，Go最近專門引入了這些型別的語言特性來支援大規模重構（參見https://talks.golang.org/2016/refactor.article).
```

-----

##### Case Study: Operation RoseHub  案例研究： Operation RoseHub 

LSCs have become a large part of Google’s internal culture, but they are starting to have implications in the broader world. Perhaps the best known case so far was “[Operation RoseHub](https://oreil.ly/txtDj).”

LSC已經成為谷歌內部文化的一個重要部分，但它們開始在更廣泛的世界中產生影響。迄今為止，最著名的案例也許是"Operation RoseHub"。

In early 2017, a vulnerability in the Apache Commons library allowed any Java application with a vulnerable version of the library in its transitive classpath to become susceptible to remote execution. This bug became known as the Mad Gadget. Among other things, it allowed an avaricious hacker to encrypt the San Francisco Municipal Transportation Agency’s systems and shut down its operations. Because the only requirement for the vulnerability was having the wrong library somewhere in its classpath, anything that depended on even one of many open source projects on GitHub was vulnerable.

2017年初，Apache Commons函式庫中的一個漏洞允許任何在其跨類別路徑中具有該函式庫的脆弱版本的Java應用程式變得容易被遠端執行。這個漏洞被稱為 "瘋狂小工具"。在其他方面，它允許一個貪婪的黑客對舊金山市交通局的系統進行加密並關閉其運作。由於該漏洞的唯一要求是在其classpath中的某個地方有錯誤的函式庫，任何依賴於GitHub上許多開源專案的東西都會受到攻擊。

To solve this problem, some enterprising Googlers launched their own version of the LSC process. By using tools such as [BigQuery](https://cloud.google.com/bigquery), volunteers identified affected projects and sent more than 2,600 patches to upgrade their versions of the Commons library to one that addressed Mad Gadget. Instead of automated tools managing the process, more than 50 humans made this LSC work.

為了解決這個問題，一些有進取心的Googlers發起了他們自己版本的LSC程式。透過使用[BigQuery](https://cloud.google.com/bigquery)等工具，志願者們確定了受影響的專案，併發送了2600多個補丁，將其版本的Commons函式庫升級為解決Mad Gadget的版本。在這個過程中，不是由自動化工具來管理，而是由50多個人類別來完成這個LSC的工作。

-----

## The LSC Process  LSC過程

With these pieces of infrastructure in place, we can now talk about the process for actually making an LSC. This roughly breaks down into four phases (with very nebulous boundaries between them):

1.  Authorization

2.  Change creation

3.  Shard management

4.  Cleanup

有了這些基礎設施，我們現在可以談談實際製作LSC的過程。這大致可分為四個階段（它們之間的界限非常模糊）：

- 授權

- 變更建立

- 分片管理

- 清理

Typically, these steps happen after a new system, class, or function has been written, but it’s important to keep them in mind during the design of the new system. At Google, we aim to design successor systems with a migration path from older systems in mind, so that system maintainers can move their users to the new system automatically.

通常，這些步驟發生在編寫新系統、類別或函式之後，但在設計新系統時記住它們很重要。在谷歌，我們的目標是在設計後繼系統時考慮到從舊系統的遷移路徑，以便系統維護人員能夠自動將使用者轉移到新系統。

### Authorization  授權

We ask potential authors to fill out a brief document explaining the reason for a proposed change, its estimated impact across the codebase (i.e., how many smaller shards the large change would generate), and answers to any questions potential reviewers might have. This process also forces authors to think about how they will describe the change to an engineer unfamiliar with it in the form of an FAQ and proposed change description. Authors also get “domain review” from the owners of the API being refactored.

我們要求潛在作者填寫一份簡短的文件，解釋提出變更的原因、其對整個程式碼函式庫的估計影響（即，大變更將產生多少較小的碎片），並回答潛在評審員可能提出的任何問題。這一過程還迫使作者思考他們將如何以常見問題解答和提出的變更描述的形式向不熟悉變更的工程師描述變更。作者還可以從正在重構的API的所有者那裡獲得"專業審查"。

This proposal is then forwarded to an email list with about a dozen people who have oversight over the entire process. After discussion, the committee gives feedback on how to move forward. For example, one of the most common changes made by the committee is to direct all of the code reviews for an LSC to go to a single “global approver.” Many first-time LSC authors tend to assume that local project owners should review everything, but for most mechanical LSCs, it’s cheaper to have a single expert understand the nature of the change and build automation around reviewing it properly.

然後，這個提案被轉發到一個有大約十幾個人的電子郵件列表，這些人對整個過程進行監督。經過討論，委員會就如何推進工作給出反饋。例如，委員會做出的最常見的改變之一是將一個LSC的所有程式碼審查交給一個 "全球批准人"。許多第一次做LSC的人傾向於認為當地的專案負責人應該審查所有的東西，但對於大多數自動LSC來說，讓一個專家瞭解變化的性質並圍繞著正確的審查建立自動化是比較低成本。

After the change is approved, the author can move forward in getting their change submitted. Historically, the committee has been very liberal with their approval,[13](#_bookmark2005) and often gives approval not just for a specific change, but also for a broad set of related changes. Committee members can, at their discretion, fast-track obvious changes without the need for full deliberation.

在修改被批准後，作者可以繼續推進他們的修改提交。從歷史上看，委員會在批准方面是非常寬鬆的，而且常常不僅批准某一特定的修改，而且批准一系列廣泛的相關修改。委員會成員可以酌情快速處理明顯的修改，而不需要進行充分的審議。

The intent of this process is to provide oversight and an escalation path, without being too onerous for the LSC authors. The committee is also empowered as the escalation body for concerns or conflicts about an LSC: local owners who disagree with the change can appeal to this group who can then arbitrate any conflicts. In practice, this has rarely been needed.

這個過程的目的是提供監督和升級的途徑，而不對LSC的作者過於繁瑣。該委員會還被授權作為對LSC的擔憂或衝突的升級機構：不同意改變的本地業主可以向該小組提出上訴，該小組可以對任何衝突進行仲裁。在實踐中，很少需要這樣做。

```
13  The only kinds of changes that the committee has outright rejected have been those that are deemed dangerous, such as converting all NULL instances to nullptr, or extremely low-value, such as changing spelling from British English to American English, or vice versa. As our experience with such changes has increased and the cost of LSCs has dropped, the threshold for approval has as well.

13  委員會完全拒絕的唯一型別的更改是那些被視為危險的更改，如將所有空實例轉換為空PTR，或極低的值，如將拼寫從英式英語更改為美式英語，或反之亦然。隨著我們在此類別變更方面的經驗增加，LSC的成本降低，批准門檻也隨之降低。
```

### Change Creation 變更建立

After getting the required approval, an LSC author will begin to produce the actual code edits. Sometimes, these can be generated comprehensively into a single large global change that will be subsequently sharded into many smaller independent pieces. Usually, the size of the change is too large to fit in a single global change, due to technical limitations of the underlying version control system.

在獲得必要的批准後，LSC作者將開始製作實際的程式碼編輯。有時，這些內容可以全面地產生一個大的全域性變化，隨後將被分割成許多小的獨立部分。通常情況下，由於底層版本控制系統的技術限制，修改的規模太大，無法容納在一個全域性修改中。

The change generation process should be as automated as possible so that the parent change can be updated as users backslide into old uses[14](#_bookmark2009) or textual merge conflicts occur in the changed code. Occasionally, for the rare case in which technical tools aren’t able to generate the global change, we have sharded change generation across humans (see [“Case Study: Operation RoseHub” on page 472](#_bookmark1994)). Although much more labor intensive than automatically generating changes, this allows global changes to happen much more quickly for time-sensitive applications.

變更產生過程應儘可能自動化，以便在使用者退回到舊的使用方式14或在變更的程式碼中出現文字合併衝突時，可以更新父級變更。偶爾，在技術工具無法產生全域性變更的罕見情況下，我們也會將變更的產生分給人工（見第472頁的 "案例研究：RoseHub行動"）。儘管這比自動產生變更要耗費更多的人力，但對於時間敏感的應用來說，這使得全域性性的變更能夠更快發生。

Keep in mind that we optimize for human readability of our codebase, so whatever tool generates changes, we want the resulting changes to look as much like humangenerated changes as possible. This requirement leads to the necessity of style guides and automatic formatting tools (see [Chapter 8](#_bookmark580)).[15](#_bookmark2010)

請記住，我們對程式碼函式庫的可讀性進行了優化，所以無論什麼工具產生的變化，我們都希望產生的變化看起來儘可能的像人類產生的變更。這一要求導致了風格指南和自動格式化工具的必要性（見第8章）。

### Sharding and Submitting  分區與提交

After a global change has been generated, the author then starts running Rosie. Rosie takes a large change and shards it based upon project boundaries and ownership rules into changes that *can* be submitted atomically. It then puts each individually sharded change through an independent test-mail-submit pipeline. Rosie can be a heavy user of other pieces of Google’s developer infrastructure, so it caps the number of outstanding shards for any given LSC, runs at lower priority, and communicates with the rest of the infrastructure about how much load it is acceptable to generate on our shared testing infrastructure.

在全域性變更產生之後，作者就開始執行Rosie。Rosie接收一個大的變化，並根據專案邊界和所有權規則將其分割成可以原子化提交的變化。然後，它把每個單獨的分片變化透過一個獨立的測試-郵件-提交管道。Rosie可能是谷歌開發者基礎設施其他部分的重度使用者，所以它對任何給定的LSC的未完成分片數量設定上限，以較低的優先順序執行，並與基礎設施的其他部分進行溝通，瞭解它在我們的共享測試基礎設施上產生多少負載是可以接受的。

We talk more about the specific test-mail-submit process for each shard below.

我們在下面會更多地談論每個分片的具體測試-郵件提交過程。

```
14   This happens for many reasons: copy-and-paste from existing examples, committing changes that have been in development for some time, or simply reliance on old habits.

15   In actuality, this is the reasoning behind the original work on clang-format for C++.

14  發生這種情況的原因有很多：從現有示例複製和貼上，提交已經開發了一段時間的更改，或者僅僅依靠舊習慣。
15  實際上，這是C++ CLAN格式的原始工作背後的推理。

```

-----

##### Cattle Versus Pets  牛與寵物

We often use the “cattle and pets” analogy when referring to individual machines in a distributed computing environment, but the same principles can apply to changes within a codebase.

當提到分散式計算環境中的單個機器時，我們經常使用 "牛和寵物 "的比喻，但同樣的原則可以適用於程式碼函式庫中的變化。

At Google, as at most organizations, typical changes to the codebase are handcrafted by individual engineers working on specific features or bug fixes. Engineers might spend days or weeks working through the creation, testing, and review of a single change. They come to know the change intimately, and are proud when it is finally committed to the main repository. The creation of such a change is akin to owning and raising a favorite pet.

在谷歌，和大多陣列織一樣，程式碼函式庫的典型變化是由從事特定功能或錯誤修復的個別工程師手動產生的。工程師們可能會花幾天或幾周的時間來建立、測試和審查一個單一的變化。他們密切瞭解這個變化，當它最終被提交到主資源函式庫時，他們會感到很自豪。建立這樣的變化就像擁有和養育一隻喜愛的寵物一樣。

In contrast, effective handling of LSCs requires a high degree of automation and produces an enormous number of individual changes. In this environment, we’ve found it useful to treat specific changes as cattle: nameless and faceless commits that might be rolled back or otherwise rejected at any given time with little cost unless the entire herd is affected. Often this happens because of an unforeseen problem not caught by tests, or even something as simple as a merge conflict.

相比之下，有效地處理LSC需要高度的自動化，併產生大量的單獨變化。在這種環境下，我們發現把特定的修改當作牛來對待是很有用的：無名無姓的提交，在任何時候都可能被回滾或以其他方式拒絕，除非整個牛群受到影響，否則代價很小。通常情況下，這種情況發生的原因是測試沒有發現的意外問題，甚至是像合併衝突這樣簡單的事情。

With a “pet” commit, it can be difficult to not take rejection personally, but when working with many changes as part of a large-scale change, it’s just the nature of the job. Having automation means that tooling can be updated and new changes generated at very low cost, so losing a few cattle now and then isn’t a problem.

對於一個 "寵物 "提交，不把拒絕放在心上是很難的，但當作為大規模變革的一部分而處理許多變化時，這只是工作的性質。擁有自動化意味著工具可以更新，並以非常低的成本產生新的變化，所以偶爾失去幾頭牛並不是什麼問題。

-----

#### Testing 測試

Each independent shard is tested by running it through TAP, Google’s CI framework. We run every test that depends on the files in a given change transitively, which often creates high load on our CI system.

每個獨立的分片都是透過谷歌的CI框架TAP來測試的。我們執行每一個依賴於特定變化中的檔案的測試，這常常給我們的CI系統帶來高負荷。

This might sound computationally expensive, but in practice, the vast majority of shards affect fewer than one thousand tests, out of the millions across our codebase. For those that affect more, we can group them together: first running the union of all affected tests for all shards, and then for each individual shard running just the intersection of its affected tests with those that failed the first run. Most of these unions cause almost every test in the codebase to be run, so adding additional changes to that batch of shards is nearly free.

這可能聽起來很昂貴，但實際上，在我們的程式碼函式庫中的數百萬個測試中，絕大多數碎片影響的測試不到一千。對於那些影響更多的測試，我們可以將它們分組：首先執行所有分片的所有受影響測試的聯合，然後對於每個單獨的分片，只執行其受影響的測試與那些第一次執行失敗的測試的交集。這些聯合體中的大多數導致程式碼函式庫中的幾乎每一個測試都被執行，因此向該批分片新增額外的變化幾乎是無額外負擔的。

One of the drawbacks of running such a large number of tests is that independent low-probability events are almost certainties at large enough scale. Flaky and brittle tests, such as those discussed in [Chapter 11](#_bookmark838), which often don’t harm the teams that write and maintain them, are particularly difficult for LSC authors. Although fairly low impact for individual teams, flaky tests can seriously affect the throughput of an LSC system. Automatic flake detection and elimination systems help with this issue, but it can be a constant effort to ensure that teams that write flaky tests are the ones that bear their costs.

執行如此大量的測試的缺點之一是，獨立的低概率事件在足夠大的規模下幾乎是確定出現的。脆弱和易碎的測試，如第11章中討論的那些，通常不會損害編寫和維護它們的團隊，對LSC作者來說特別困難。雖然對單個團隊的影響相當小，但分片測試會嚴重影響LSC系統的吞吐量。自動片斷檢測和消除系統有助於解決這個問題，但要確保編寫片斷測試的團隊承擔其成本，這可能是一個持續的努力。

In our experience with LSCs as semantic-preserving, machine-generated changes, we are now much more confident in the correctness of a single change than a test with any recent history of flakiness—so much so that recently flaky tests are now ignored when submitting via our automated tooling. In theory, this means that a single shard can cause a regression that is detected only by a flaky test going from flaky to failing. In practice, we see this so rarely that it’s easier to deal with it via human communication rather than automation.

根據我們對LSC作為語義保護、機器產生的更改的經驗，我們現在對單個變化的正確性比對近期有任何不穩定測試更有信心--以至於最近不穩定測試現在在透過我們的自動化工具提交時被忽略了。在理論上，這意味著一個單一的分片可能會導致迴歸，而這個迴歸只能由一個不穩定的測試從不穩定到失敗來檢測。在實踐中，我們很少看到這種情況，所以透過人工溝通而不是自動化來處理它。

For any LSC process, individual shards should be committable independently. This means that they don’t have any interdependence or that the sharding mechanism can group dependent changes (such as to a header file and its implementation) together. Just like any other change, large-scale change shards must also pass project-specific checks before being reviewed and committed.

對於任何LSC過程來說，各個分片應該是可以獨立提交的。這意味著它們沒有任何相互依賴性，或者說分片機制可以將相互依賴的變更（比如對標頭檔案和其實現的變更）歸為一組。就像其他變化一樣，大規模的更改分片在被審查和提交之前也必須透過專案特定的檢查。

#### Mailing reviewers  推送審稿人

After Rosie has validated that a change is safe through testing, it mails the change to an appropriate reviewer. In a company as large as Google, with thousands of engineers, reviewer discovery itself is a challenging problem. Recall from [Chapter 9 ](#_bookmark664)that code in the repository is organized with OWNERS files, which list users with approval privileges for a specific subtree in the repository. Rosie uses an owners detection service that understands these OWNERS files and weights each owner based upon their expected ability to review the specific shard in question. If a particular owner proves to be unresponsive, Rosie adds additional reviewers automatically in an effort to get a change reviewed in a timely manner.

在Rosie透過測試驗證了某項變更是安全的之後，它就會將該變更推送給適當的審查員。在谷歌這樣一個擁有數千名工程師的大公司，審查員的發現本身就是一個具有挑戰性的問題。回顧第九章，版本函式庫中的程式碼是用OWNERS檔案組織的，這些檔案列出了對版本函式庫中特定子樹有批准許可權的使用者。Rosie使用一個所有者檢測服務來理解這些OWNERS檔案，並根據他們審查特定分片的預期能力來衡量每個所有者。如果一個特定的所有者被證明是沒有響應的，Rosie會自動新增額外的審查者，以努力使一個變化得到及時的審查。

As part of the mailing process, Rosie also runs the per-project precommit tools, which might perform additional checks. For LSCs, we selectively disable certain checks such as those for nonstandard change description formatting. Although useful for individual changes on specific projects, such checks are a source of heterogeneity across the codebase and can add significant friction to the LSC process. This heterogeneity is a barrier to scaling our processes and systems, and LSC tools and authors can’t be expected to understand special policies for each team.

作為推送過程的一部分，Rosie也執行每個專案的預提交工具，這可能會執行額外的檢查。對於LSC，我們有選擇地禁用某些檢查，例如對非標準的修改描述格式的檢查。儘管這種檢查對於特定專案的個別更改很有用，但它是整個程式碼函式庫中異構性的一個來源，並且會給LSC過程增加很大的阻力。這種異質性是擴充套件我們流程和系統的障礙，不能指望LSC工具和作者瞭解每個團隊的特殊策略。

We also aggressively ignore presubmit check failures that preexist the change in question. When working on an individual project, it’s easy for an engineer to fix those and continue with their original work, but that technique doesn’t scale when making LSCs across Google’s codebase. Local code owners are responsible for having no preexisting failures in their codebase as part of the social contract between them and infrastructure teams.

我們還積極地忽略了預先存在問題變更的提交前檢查失敗。在處理單個專案時，工程師很容易修復這些問題並繼續他們原來的工作，但當在Google的程式碼函式庫中製作LSC時，這種技術無法擴充套件。原生代碼所有者有責任確保其程式碼函式庫中沒有先前存在的故障，這是他們與基礎設施團隊之間契約的一部分。

#### Reviewing 審查

As with other changes, changes generated by Rosie are expected to go through the standard code review process. In practice, we’ve found that local owners don’t often treat LSCs with the same rigor as regular changes—they trust the engineers generating LSCs too much. Ideally these changes would be reviewed as any other, but in practice, local project owners have come to trust infrastructure teams to the point where these changes are often given only cursory review. We’ve come to only send changes to local owners for which their review is required for context, not just approval permissions. All other changes can go to a “global approver”: someone who has ownership rights to approve *any* change throughout the repository.

與其他更改一樣，由Rosie產生的更改預計將透過標準程式碼審查過程。在實踐中，我們發現本地業主通常不會像對待普通變更那樣嚴格對待LSC--他們太信任產生LSC的工程師了。理想情況下，這些更改會像其他更改一樣被審查，但在實踐中，本地專案業主已經開始信任基礎設施團隊，以至於這些修改往往只被粗略地審查。我們已經開始只把那些需要他們審查的變更傳送給本地所有者，而不僅僅是批准許可權。所有其他的修改都可以交給 "全域性審批人"：擁有所有權的人可以批准整個版本函式庫的任何修改。

When using a global approver, all of the individual shards are assigned to that person, rather than to individual owners of different projects. Global approvers generally have specific knowledge of the language and/or libraries they are reviewing and work with the large-scale change author to know what kinds of changes to expect. They know what the details of the change are and what potential failure modes for it might exist and can customize their workflow accordingly.

當使用全域性審批人時，所有的單個分片都被分配給這個人，而不是分配給不同專案的單個所有者。全域性審批人通常對他們正在審查的語言和/或函式庫有特定的知識，並與大規模的變更作者合作，以瞭解預期的變更型別。他們知道變化的細節是什麼，以及它可能存在的潛在失敗模式，並可以相應地訂製他們的工作流程。

Instead of reviewing each change individually, global reviewers use a separate set of pattern-based tooling to review each of the changes and automatically approve ones that meet their expectations. Thus, they need to manually examine only a small subset that are anomalous because of merge conflicts or tooling malfunctions, which allows the process to scale very well.

全域性審閱者使用一組單獨的基於模式的工具來審閱每個更改，並自動批准滿足其期望的更改，而不是單獨審閱每個更改。因此，他們只需要手動檢查一小部分由於合併衝突或工具故障而異常的子集合，這使得流程能夠很好地擴充套件。

#### Submitting 提交

Finally, individual changes are committed. As with the mailing step, we ensure that the change passes the various project precommit checks before actually finally being committed to the repository.

最後，提交單個更改。與推送步驟一樣，我們確保更改在最終提交到儲存函式庫之前透過各種專案預提交檢查。

With Rosie, we are able to effectively create, test, review, and submit thousands of changes per day across all of Google’s codebase and have given teams the ability to effectively migrate their users. Technical decisions that used to be final, such as the name of a widely used symbol or the location of a popular class within a codebase, no longer need to be final.

有了Rosie，我們能夠在谷歌的所有程式碼函式庫中有效地建立、測試、審查和提交每天數以千計的更改，並使團隊有能力有效地遷移他們的使用者。過去的技術決定，如一個廣泛使用的符號的名稱或一個流行的類別在程式碼函式庫中的位置，不再需要是最終決定。

### Cleanup  清理

Different LSCs have different definitions of “done,” which can vary from completely removing an old system to migrating only high-value references and leaving old ones to organically disappear.[16](#_bookmark2019) In almost all cases, it’s important to have a system that prevents additional introductions of the symbol or system that the large-scale change worked hard to remove. At Google, we use the Tricorder framework mentioned in Chapters [20 ](#_bookmark1781)and [19 ](#_bookmark1719)to flag at review time when an engineer introduces a new use of a deprecated object, and this has proven an effective method to prevent backsliding. We talk more about the entire deprecation process in [Chapter 15](#_bookmark1319).

不同的LSC對 "完成 "有不同的定義，從完全刪除舊系統到只遷移高價值的參考，讓舊系統有機地消失。在幾乎所有情況下，重要的是，要有一個系統，防止大規模變革努力消除的符號或系統的額外引入。在谷歌，我們使用和章節中提到的Tricorder框架，在工程師引入被廢棄物件的新用途時，在審查時進行標記，這已被證明是防止倒退的有效方法。我們在[第15章](#_bookmark1319)中更多地討論了整個廢棄過程。

## Conclusion  總結

LSCs form an important part of Google’s software engineering ecosystem. At design time, they open up more possibilities, knowing that some design decisions don’t need to be as fixed as they once were. The LSC process also allows maintainers of core infrastructure the ability to migrate large swaths of Google’s codebase from old systems, language versions, and library idioms to new ones, keeping the codebase consistent, spatially and temporally. And all of this happens with only a few dozen engineers supporting tens of thousands of others.

LSC是谷歌軟體工程生態系統的重要組成部分。在設計時，他們開啟了更多的可能性，知道一些設計決策不需要像以前那樣固定。LSC過程還允許核心基礎設施的維護者有能力將谷歌的大量程式碼函式庫從舊的系統、語言版本和函式庫習語遷移到新的系統，使程式碼函式庫在空間上和時間上保持一致。而這一切都發生在只有幾十名工程師支援數萬名其他工程師的情況下。

No matter the size of your organization, it’s reasonable to think about how you would make these kinds of sweeping changes across your collection of source code. Whether by choice or by necessity, having this ability will allow greater flexibility as your organization scales while keeping your source code malleable over time.

無論你的組織有多大的規模，你都有理由考慮如何在你的原始碼集合中進行這類別全面的改變。不管是出於選擇還是需要，擁有這種能力將使你的組織在擴大規模時有更大的靈活性，同時使你的原始碼隨著時間的推移保持可塑性。

## TL;DRs  內容提要

- An LSC process makes it possible to rethink the immutability of certain technical decisions.

- Traditional models of refactoring break at large scales.

- Making LSCs means making a habit of making LSCs.

- LSC過程可以重新思考某些技術決策的不變性。

- 重構的傳統模型在大範圍內被打破。

- 製作LSC意味著養成製作LSC的習慣。


