

**CHAPTER 15**

# Deprecation

# 第十五章 廢棄

本章由海魯姆·賴特撰寫，由湯姆·曼什雷克編輯

I love deadlines. I like the whooshing sound they make as they fly by. —Douglas Adams

我喜歡萬事都有一個截止日期。 我喜歡它們飛過時發出的嗖嗖聲。

-道格拉斯·亞當斯 如是說。

All systems age. Even though software is a digital asset and the physical bits themselves don’t degrade, new technologies, libraries, techniques, languages, and other environmental changes over time render existing systems obsolete. Old systems require continued maintenance, esoteric expertise, and generally more work as they diverge from the surrounding ecosystem. It’s often better to invest effort in turning off obsolete systems, rather than letting them lumber along indefinitely alongside the systems that replace them. But the number of obsolete systems still running suggests that, in practice, doing so is not trivial. We refer to the process of orderly migration away from and eventual removal of obsolete systems as deprecation.

所有系統都會老化。 雖說軟體是一種數字資產,它的位元組位本身不會有任何退化。但隨著時間的推移，新技術、函式庫、 技術、語言和其他環境變化,都有可能使現有的系統過時。 舊系統需要持續維護、深奧的專業知識，通常需要花費 更多的精力，因為它們與周遭的生態略有不同。 投入些精力廢棄掉過時的系統通常是個不錯的選項，讓它們無限 期地與它的替代者共存通常不是明智的選擇。 從實踐的角度出發，那些仍在執行的大量過時系統無不表明，廢棄 掉過時系統所帶來的收益並非微不足道。 我們將有序遷移並最終移除過時系統的過程稱為棄用。

Deprecation is yet another topic that more accurately belongs to the discipline of software engineering than programming because it requires thinking about how to manage a system over time. For long-running software ecosystems, planning for and executing deprecation correctly reduces resource costs and improves velocity by removing the redundancy and complexity that builds up in a system over time. On the other hand, poorly deprecated systems may cost more than leaving them alone. While deprecating systems requires additional effort, it’s possible to plan for deprecation during the design of the system so that it’s easier to eventually decommission and remove it. Deprecations can affect systems ranging from individual function calls to entire software stacks. For concreteness, much of what follows focuses on code-level deprecations.

“棄用”,嚴格意義上說,它不是一個開發層面的議題,而應歸類於軟體工程學範疇。因為它需要考慮如何隨著時間的 推移來管理系統。 對於長期執行的軟體生態系統，正確規劃執行“棄用”,可以透過消除系統中隨時間累積產生的冗 餘、複雜性等,來降低資源成本並提高速度。 另一方面，不推薦使用的系統可能比不理會它們的成本更高。 雖然 “棄用”系統需要額外花費精力，但可以考慮在系統設計期間有計劃地“棄用”，可以更容易地實現徹底停用並刪除廢棄的 系統。 “棄用”的影響範圍可大可小，小到單個函式，大到整個軟體生態。 接下來的大部分內容我們都將集中在代 碼層級“棄用”上。

Unlike with most of the other topics we have discussed in this book, Google is still learning how best to deprecate and remove software systems. This chapter describes the lessons we’ve learned as we’ve deprecated large and heavily used internal systems. Sometimes, it works as expected, and sometimes it doesn’t, but the general problem of removing obsolete systems remains a difficult and evolving concern in the industry.

與我們在本書中討論的大多數章節不同，Google 仍在學習如何最好地“棄用”和刪除軟體系統。 本章主要介紹我們在 “棄用”大型和大量使用的內部系統時學到的經驗教訓。 有時，它能符合預期，有時則不會。畢竟移除過時系統的普 遍問題,仍然是行業中一個困難且須不斷探索的問題。

This chapter primarily deals with deprecating technical systems, not end-user products. The distinction is somewhat arbitrary given that an external-facing API is just another sort of product, and an internal API may have consumers that consider themselves end users. Although many of the principles apply to turning down a public product, we concern ourselves here with the technical and policy aspects of deprecating and removing obsolete systems where the system owner has visibility into its use.

本章主要從技術層面講“棄用”，而不是從產品層面。 考慮到面向外部的 API 也算另一種產品，而內部 API 通常 是自產自銷，因此這種區別有些武斷。 儘管許多原則也適用於對外產品，但我們在這裡關注的是“棄用”和刪除過時 的內部系統的技術和策略方面的問題。



## 為什麼要“棄用” (Why Deprecate?)

Our discussion of deprecation begins from the fundamental premise that code is a liability, not an asset. After all, if code were an asset, why should we even bother spending time trying to turn down and remove obsolete systems? Code has costs, some of which are borne in the process of creating a system, but many other costs are borne as a system is maintained across its lifetime. These ongoing costs, such as the operational resources required to keep a system running or the effort to continually update its codebase as surrounding ecosystems evolve, mean that it’s worth evaluating the trade-offs between keeping an aging system running or working to turn it down.

我們對“棄用”的討論始於這樣一個基本前提，即程式碼是一種負債，而不是一種資產。 畢竟，如果程式碼是一種資產， 我們為什麼還要費心去嘗試“棄用”它呢？ 程式碼有成本，其中有開發成本，但更多的是維護成本。 這些持續的成本， 例如保持系統執行所需的運營資源或緊跟周圍生態而不斷更新迭代花費的精力，意味著你需要在繼續維護老化的系統執行和將其下線之間做一個權衡。

The age of a system alone doesn’t justify its deprecation. A system could be finely crafted over several years to be the epitome of software form and function. Some software systems, such as the LaTeX typesetting system, have been improved over the course of decades, and even though changes still happen, they are few and far between. Just because something is old, it does not follow that it is obsolete.

“棄用”並不能簡單地用專案的年限來定奪。 一個系統可以經過數年精心打造，才能穩定成熟。 一些軟體系統，比如 LaTeX 排版系統，經過幾十年的改進，雖然它仍在發生變化，但已經趨向穩定了。 系統老舊並不意味著它過時了。

Deprecation is best suited for systems that are demonstrably obsolete and a replacement exists that provides comparable functionality. The new system might use resources more efficiently, have better security properties, be built in a more sustainable fashion, or just fix bugs. Having two systems to accomplish the same thing might not seem like a pressing problem, but over time, the costs of maintaining them both can grow substantially. Users may need to use the new system, but still have dependencies that use the obsolete one.

“棄用”最適合那些明顯過時的系統，並且存在提供類似功能的替代品。 新系統可能更有效地使用資源，具有更好的 安全屬性，以更可持續的方式建構，或者只是修復錯誤。 擁有兩個系統來完成同一件事似乎不是一個緊迫的問題， 但隨著時間的推移，維護它們的成本會大幅增加。 使用者可能需要使用新系統，但仍然依賴於使用過時的系統。

with the old one. Spending the effort to remove the old system can pay off as the replacement system can now evolve more quickly. The two systems might need to interface with each other, requiring complicated transformation code. As both systems evolve, they may come to depend on each other, making eventual removal of either more difficult. In the long run, we’ve discovered that having multiple systems performing the same function also impedes the evolution of the newer system because it is still expected to maintain compatibility with the old one. Spending the effort to remove the old system can pay off as the replacement system can now evolve more quickly.

這兩個系統可能需要相互連線，需要複雜的轉換程式碼。 隨著這兩個系統的發展，它們可能會相互依賴，從而使最 終消除其中任何一個變得更加困難。 從長遠來看，我們發現讓多個系統執行相同的功能也會阻礙新系統的發展， 因為它仍然需要與舊的保持相容性。 由於替換系統現在可以更快地發展，因此花費精力移除舊系統會有相關的收 益。

> Earlier we made the assertion that “code is a liability, not an asset.” If that is true, why have we spent most of this book discussing the most efficient way to build software systems that can live for decades? Why put all that effort into creating more code when it’s simply going to end up on the liability side of the balance sheet?
> 
> Code itself doesn’t bring value: it is the functionality that it provides that brings value. That functionality is an asset if it meets a user need: the code that implements this functionality is simply a means to that end. If we could get the same functionality from a single line of maintainable, understandable code as 10,000 lines of convoluted spaghetti code, we would prefer the former. Code itself carries a cost—the simpler the code is, while maintaining the same amount of functionality, the better.
> 
> Instead of focusing on how much code we can produce, or how large is our codebase, we should instead focus on how much functionality it can deliver per unit of code and try to maximize that metric. One of the easiest ways to do so isn’t writing more code and hoping to get more functionality; it’s removing excess code and systems that are no longer needed. Deprecation policies and procedures make this possible.
> 
> 前面，我們斷言“程式碼是一種負債，而不是一種資產”。如果這是真的，為什麼我們用本書的大部分時間來討論建構 可以存活數十年的軟體系統的最有效方法？當它最終會出現在資產負債表的負債方時，為什麼還要付出所有努力來 建立更多程式碼呢？程式碼本身不會帶來價值：它提供的功能帶來了價值。如果該功能滿足使用者需求，那麼它就是一種 資產：實現此功能的程式碼只是實現該目的的一種手段。如果我們可以從一行可維護、可理解的程式碼中獲得與 10,000 行錯綜複雜的意大利麵條式程式碼相同的功能，我們會更喜歡前者。程式碼本身是有成本的——程式碼越簡單，同 時保持相同數量的功能越好。與其關注我們可以生產多少程式碼，或者我們的程式碼函式庫有多大，我們應該關注每單位代 碼可以提供多少功能，並嘗試最大化該指標。最簡單的方法之一就是不要編寫更多程式碼並希望獲得更多功能；而是 刪除不再需要的多餘程式碼和系統。“棄用”策略的存在就是為了解決這個問題。

Even though deprecation is useful, we’ve learned at Google that organizations have a limit on the amount of deprecation work that is reasonable to undergo simultaneously, from the aspect of the teams doing the deprecation as well as the customers of those teams. For example, although everybody appreciates having freshly paved roads, if the public works department decided to close down every road for paving simultaneously, nobody would go anywhere. By focusing their efforts, paving crews can get specific jobs done faster while also allowing other traffic to make progress. Likewise, it’s important to choose deprecation projects with care and then commit to following through on finishing them.

儘管“棄用”很有用，但我們在 Google 瞭解到，從執行“棄用”的團隊以及這些團隊的客戶的角度來看，對同時進行的 ““棄用””是有數量上的限制的。 例如，雖然每個人都喜歡新鋪設的道路，但如果政府部門決定同時關閉所有道路並 進行鋪設，那麼將會導致大家無路可走。 透過集中精力，鋪設人員可以更快地完成特定工作，但同時不應該影響 其他道路的通行。 故同樣重要的是要謹慎選擇“棄用”專案並付諸實施。

## 為什麼“棄用”這麼難(Why Is Deprecation So Hard?)

We’ve mentioned Hyrum’s Law elsewhere in this book, but it’s worth repeating its applicability here: the more users of a system, the higher the probability that users are using it in unexpected and unforeseen ways, and the harder it will be to deprecate and remove such a system. Their usage just “happens to work” instead of being “guaranteed to work.” In this context, removing a system can be thought of as the ultimate change: we aren’t just changing behavior, we are removing that behavior completely! This kind of radical alteration will shake loose a number of unexpected dependents.

我們在本書的其他地方提到了海拉姆定律，但值得在這裡重申一下它的適用性：一個系統的使用者越多，使用者以意外 和不可預見的方式使用它的可能性就越大，並且越難“棄用”和刪除這樣的系統。 它們有可能只是“碰巧可用”而不是 “絕對可用”。 在這種情況下，“棄用”不是簡單的行為變更，而是一次大變革-徹底的“棄用”！ 這種激進的改變可能會 對這樣的系統造成意想不到的影響。

To further complicate matters, deprecation usually isn’t an option until a newer system is available that provides the same (or better!) functionality. The new system might be better, but it is also different: after all, if it were exactly the same as the obsolete system, it wouldn’t provide any benefit to users who migrate to it (though it might benefit the team operating it). This functional difference means a one-to-one match between the old system and the new system is rare, and every use of the old system must be evaluated in the context of the new one.

更復雜的是，在提供相同（或更好）功能的新系統可用之前，“棄用”通常不是一種選擇。 新系統可能更好,但也有不 同：畢竟，如果它和過時的系統完全一樣，它不會為遷移到它的使用者提供任何好處（儘管它可能使執行它的團隊 受益）。 這種功能差異意味著舊系統和新系統之間的一對一匹配很少見，新老系統的切換通常需要進行評估。

Another surprising reluctance to deprecate is emotional attachment to old systems, particularly those that the deprecator had a hand in helping to create. An example of this change aversion happens when systematically removing old code at Google: we’ve occasionally encountered resistance of the form “I like this code!” It can be difficult to convince engineers to tear down something they’ve spent years building. This is an understandable response, but ultimately self-defeating: if a system is obsolete, it has a net cost on the organization and should be removed. One of the ways we’ve addressed concerns about keeping old code within Google is by ensuring that the source code repository isn’t just searchable at trunk, but also historically. Even code that has been removed can be found again (see Chapter 17).

另一個令人驚訝的不願“棄用”的現象是對舊系統的情感依戀，尤其是那些“棄用”者幫助建立的系統。 在 Google 系統 地刪除舊程式碼時，就會發生這種厭惡更改的一個例子：我們偶爾會遇到“我喜歡這段程式碼！”這種形式的抵制。 說 服工程師刪除他們花了多年時間建造的東西可能很困難。 這是一種可以理解的反應，但最終會弄巧成拙：如果一 個系統已經過時，它會給組織帶來淨成本，應該將其刪除。 我們解決了將舊程式碼保留在 Google 中的問題的方法 之一是確保原始碼儲存函式庫不僅可以在主幹上搜索，而且可以在歷史上搜尋。 甚至被刪除的程式碼也能再次找到 (見 17章)

> There’s an old joke within Google that there are two ways of doing things: the one that’s deprecated, and the one that’s not-yet-ready. This is usually the result of a new solution being “almost” done and is the unfortunate reality of working in a technological environment that is complex and fast-paced.
> 
> Google engineers have become used to working in this environment, but it can still be disconcerting. Good documentation, plenty of signposts, and teams of experts helping with the deprecation and migration process all make it easier to know whether you should be using the old thing, with all its warts, or the new one, with all its uncertainties.
> 
> 谷歌內部有一個古老的笑話，說有兩種做事方式：一種已被“棄用”，另一種尚未準備就緒。 這通常發生成新解決方 案“幾乎”完成的時候，並且是在複雜且快節奏的技術環境中工作的不幸現實。谷歌工程師已經習慣了在這種環境中 工作，但它仍然令人不安。 良好的文件、大量的指引以及幫助“棄用”和遷移過程的專家團隊,都可以讓您更容易地判 斷是使用舊的，有缺點，還是新的，有不確定性的。

Finally, funding and executing deprecation efforts can be difficult politically; staffing a team and spending time removing obsolete systems costs real money, whereas the costs of doing nothing and letting the system lumber along unattended are not readily observable. It can be difficult to convince the relevant stakeholders that deprecation efforts are worthwhile, particularly if they negatively impact new feature development. Research techniques, such as those described in Chapter 7, can provide concrete evidence that a deprecation is worthwhile.

最後，資助和執行“棄用”工作在政治上可能很困難； 為團隊配備人員並花時間移除過時的系統會花費大量金錢，而 無所作為和讓系統在無人看管的情況下緩慢執行的成本不易觀察到。 很難讓相關利益相關者相信“棄用”工作是值得 的，尤其是當它們對新功能開發產生負面影響時。 研究技術，例如第 7 章中描述的那些，可以提供具體的證據證 明“棄用”是值得的。

Given the difficulty in deprecating and removing obsolete software systems, it is often easier for users to evolve a system in situ, rather than completely replacing it. Incrementality doesn’t avoid the deprecation process altogether, but it does break it down into smaller, more manageable chunks that can yield incremental benefits. Within Google, we’ve observed that migrating to entirely new systems is extremely expensive, and the costs are frequently underestimated. Incremental deprecation efforts accomplished by in-place refactoring can keep existing systems running while making it easier to deliver value to users.

鑑於“棄用”和刪除過時軟體系統的難度，使用者通常更容易就地改進系統，而不是完全替換它。 增量並沒有完全避免 “棄用”過程，但它確實將其分解為更小、更易於管理的塊，這些塊可以產生增量收益。 在 Google 內部，我們觀察 到遷移到全新系統的成本非常高，而且成本經常被低估。 增量“棄用”工作透過就地重構實現的功能可以保持現有系 統執行，同時更容易向用戶交付價值。


### 設計之初便考慮“棄用” (Deprecation During Design)

Like many engineering activities, deprecation of a software system can be planned as those systems are first built. Choices of programming language, software architecture, team composition, and even company policy and culture all impact how easy it will be to eventually remove a system after it has reached the end of its useful life.

與許多工程活動一樣，軟體系統的“棄用”可以在這些系統首次設計時便進行規劃。 程式語言、軟體架構、團隊組成， 甚至公司策略和文化的選擇都會影響系統在使用壽命結束後最終將其“棄用”的難易程度。

The concept of designing systems so that they can eventually be deprecated might be radical in software engineering, but it is common in other engineering disciplines. Consider the example of a nuclear power plant, which is an extremely complex piece of engineering. As part of the design of a nuclear power station, its eventual decommissioning after a lifetime of productive service must be taken into account, even going so far as to allocate funds for this purpose.[^1] Many of the design choices in building a nuclear power plant are affected when engineers know that it will eventually need to be decommissioned.

設計系統以使其最終可以被“棄用”的概念在軟體工程中可能是激進的，但它在其他工程學科中很常見。 以核電站為 例，這是一項極其複雜的工程。 作為核電站設計的一部分，必須考慮到其在服務壽命到期後最終退役，甚至為此 分配資金。 當工程師知道它最終需要退役時，核電站建設中的許多設計,將會隨之改變。

Unfortunately, software systems are rarely so thoughtfully designed. Many software engineers are attracted to the task of building and launching new systems, not maintaining existing ones. The corporate culture of many companies, including Google, emphasizes building and shipping new products quickly, which often provides a disincentive for designing with deprecation in mind from the beginning. And in spite of the popular notion of software engineers as data-driven automata, it can be psychologically difficult to plan for the eventual demise of the creations we are working so hard to build.

不幸的是，軟體系統很少經過精心設計。 許多軟體工程師更熱心於建構和啟動新系統，而不是維護現有系統。 包 括 Google 在內的許多公司的企業文化都強調快速建構和交付新產品，這通常會阻礙從一開始就考慮“棄用”的設計。 儘管普遍認為軟體工程師是資料驅動的自動機，但在心理上很難為我們辛勤工作的創造物的最終消亡做計劃。

So, what kinds of considerations should we think about when designing systems that we can more easily deprecate in the future? Here are a couple of the questions we encourage engineering teams at Google to ask: 

- How easy will it be for my consumers to migrate from my product to a potential replacement?
- How can I replace parts of my system incrementally?

那麼，在設計我們將來更容易“棄用”的系統時，我們應該考慮哪些因素？ 以下是我們鼓勵 Google 的工程團隊提出的幾個問題：

-   我的使用者從我的產品遷移到潛在替代品的難易程度如何？
-   如何逐步更換系統部件？

Many of these questions relate to how a system provides and consumes dependencies. For a more thorough discussion of how we manage these dependencies, see Chapter 16.

其中許多問題與系統如何提供和使用依賴項有關。 有關我們如何管理這些依賴項的更深入討論，請參閱第 16 章。

Finally, we should point out that the decision as to whether to support a project long term is made when an organization first decides to build the project. After a software system exists, the only remaining options are support it, carefully deprecate it, or let it stop functioning when some external event causes it to break. These are all valid options, and the trade-offs between them will be organization specific. A new startup with a single project will unceremoniously kill it when the company goes bankrupt, but a large company will need to think more closely about the impact across its portfolio and reputation as they consider removing old projects. As mentioned earlier, Google is still learning how best to make these trade-offs with our own internal and external products.

最後，我們應該指出，是否長期支援專案的決定,是在組織最初決定建立專案時做出的。軟體系統存在後，剩下的 唯一選擇是支援它，小心地“棄用”它，或者在某些外部事件導致它崩潰時讓它停止執行。這些都是有效的選項，它們 之間的權衡將是特定於組織的。當公司破產時，一個只有一個專案的新創業公司會毫不客氣地殺死它，但一家大公 司在考慮刪除舊專案時需要更仔細地考慮對其投資組合和聲譽的影響。如前所述，谷歌仍在學習如何最好地利用我 們自己的內部和外部產品進行這些權衡。

In short, don’t start projects that your organization isn’t committed to support for the expected lifespan of the organization. Even if the organization chooses to deprecate and remove the project, there will still be costs, but they can be mitigated through planning and investments in tools and policy.

簡而言之，如果你的公司不打算長期支援某個專案，那麼輕易不要啟動這個專案。即使公司選擇“棄用”專案，仍然會 有成本，但可以透過規劃和投資工具和策略來降低成本。

> [^1]: “Design and Construction of Nuclear Power Plants to Facilitate Decommissioning,” Technical Reports Series No. 382, IAEA, Vienna (1997)./
> 1 "設計和建造核電站便捷退役"，技術報告系列第382號，IAEA，維也納（1997年）。


## “棄用”的種類 (Types of Deprecation)

Deprecation isn’t a single kind of process, but a continuum of them, ranging from “we’ll turn this off someday, we hope” to “this system is going away tomorrow, customers better be ready for that.” Broadly speaking, we divide this continuum into two separate areas: advisory and compulsory.

“棄用”不是一種單一的過程，而是一個連續的過程，從“我們希望有一天會關閉它”到“這個系統明天就會消失，客戶 最好為此做好準備。” 從廣義上講，我們將這個連續統一體分為兩個獨立的領域：建議和強制。

### 建議性“棄用” (Advisory Deprecation)

Advisory deprecations are those that don’t have a deadline and aren’t high priority for the organization (and for which the company isn’t willing to dedicate resources). These could also be labeled aspirational deprecations: the team knows the system has been replaced, and although they hope clients will eventually migrate to the new system, they don’t have imminent plans to either provide support to help move clients or delete the old system. This kind of deprecation often lacks enforcement: we hope that clients move, but can’t force them to. As our friends in SRE will readily tell you: “Hope is not a strategy.”

建議性“棄用”是那些沒有截止日期並且對組織來說不是高優先順序的（並且公司不願意為此投入資源）。這些也可能被標 記為理想“棄用”：團隊知道系統已被替換，儘管他們希望客戶最終遷移到新系統，但他們沒有近期的計劃來提供支援 以幫助客戶遷移或刪除舊系統。這種“棄用”往往缺乏執行力：我們希望客戶遷移，但不強迫他們做。正如我們在 SRE 的朋友會很容易告訴你的那樣：“希望不是策略。”

Advisory deprecations are a good tool for advertising the existence of a new system and encouraging early adopting users to start trying it out. Such a new system should not be considered in a beta period: it should be ready for production uses and loads and should be prepared to support new users indefinitely. Of course, any new system is going to experience growing pains, but after the old system has been deprecated in any way, the new system will become a critical piece of the organization’s infrastructure.

建議性“棄用”是宣傳新系統存在並鼓勵早期採用的使用者開始嘗試的好工具。這樣的新系統不應該在測試階段被考慮：它 應該準備好用於生產用途和負載，並且應該準備好無限期地支援新使用者。當然，任何新系統都會經歷成長的痛苦， 但是在舊系統以任何方式被“棄用”之後，新系統將成為組織基礎設施的關鍵部分。

One scenario we’ve seen at Google in which advisory deprecations have strong benefits is when the new system offers compelling benefits to its users. In these cases, simply notifying users of this new system and providing them self-service tools to migrate to it often encourages adoption. However, the benefits cannot be simply incremental: they must be transformative. Users will be hesitant to migrate on their own for marginal benefits, and even new systems with vast improvements will not gain full adoption using only advisory deprecation efforts.

我們在谷歌看到的一種情況是，當新系統為其使用者提供令人信服的好處時，建議性“棄用”具有強大的好處。 在這些情 況下，簡單地通知使用者這個新系統並為他們提供自助服務工具以遷移到它,通常會鼓勵採用。然而，收益不能簡單 地遞增：它們必須具有變革性。否則使用者將不願為了這一點點邊際收益而自行遷移，不過對於“建議性“棄用””，即 使具有巨大改進的新系統也通常不會被完全採納。

Advisory deprecation allows system authors to nudge users in the desired direction, but they should not be counted on to do the majority of migration work. It is often tempting to simply put a deprecation warning on an old system and walk away without any further effort. Our experience at Google has been that this can lead to (slightly) fewer new uses of an obsolete system, but it rarely leads to teams actively migrating away from it. Existing uses of the old system exert a sort of conceptual (or technical) pull toward it: comparatively many uses of the old system will tend to pick up a large share of new uses, no matter how much we say, “Please use the new system.” The old system will continue to require maintenance and other resources unless its users are more actively encouraged to migrate.

建議性“棄用”允許系統作者將使用者推向所需的方向，但不應指望他們完成大部分遷移工作。通常只需要在舊系統上簡 單地發出“棄用”警告，然後棄之不顧即可。我們在 Google 的經驗是，這可能會導致（略微）減少對過時系統的使用， 但很少會導致團隊積極遷移。舊系統的現有功能會有一種吸引力，吸引更多的系統使用它,無論我們說多少，​​“請使 用新的系統。”除非更積極地鼓勵其使用者遷移，否則舊系統將需要繼續維護。


### 強制性“棄用” (Compulsory Deprecation)

This active encouragement comes in the form of compulsory deprecation. This kind of deprecation usually comes with a deadline for removal of the obsolete system: if users continue to depend on it beyond that date, they will find their own systems no longer work.

這種“棄用”通常伴隨著刪除過時系統的最後期限：如果使用者在該日期之後繼續依賴它，他們將發現自己的系統不再正 常工作。

Counterintuitively, the best way for compulsory deprecation efforts to scale is by localizing the expertise of migrating users to within a single team of experts—usually the team responsible for removing the old system entirely. This team has incentives to help others migrate from the obsolete system and can develop experience and tools that can then be used across the organization. Many of these migrations can be effected using the same tools discussed in Chapter 22.

與直覺相反，推廣強制性“棄用”工作的最佳方法是將遷移使用者的工作交給一個專家團隊——通常是負責完全刪除舊系統 的團隊。該團隊有動力幫助其他人從過時的系統遷移，並可以開發可在整個組織中使用的經驗和工具。許多這些遷 移可以使用第 22 章中討論的相同工具來實現。

For compulsory deprecation to actually work, its schedule needs to have an enforcement mechanism. This does not imply that the schedule can’t change, but empower the team running the deprecation process to break noncompliant users after they have been sufficiently warned through efforts to migrate them. Without this power, it becomes easy for customer teams to ignore deprecation work in favor of features or other more pressing work.

為了讓強制性“棄用”真正起作用，需要有一個強制執行的時間表。並以警告的形式通知到需要執行遷移的客戶團隊。 沒有這種能力，客戶團隊很容易忽略“棄用”工作，而轉而支援其他更緊迫的工作。

At the same time, compulsory deprecations without staffing to do the work can come across to customer teams as mean spirited, which usually impedes completing the deprecation. Customers simply see such deprecation work as an unfunded mandate, requiring them to push aside their own priorities to do work just to keep their services running. This feels much like the “running to stay in place” phenomenon and creates friction between infrastructure maintainers and their customers. It’s for this reason that we strongly advocate that compulsory deprecations are actively staffed by a specialized team through completion.

同時，若沒有安排人員協助，可能會給客戶團隊帶來刻薄的印象，這通常會影響遷移的進度。 客戶只是將它視為 一項沒有資金的任務，要求他們擱置自己的優先事項，只為保持服務執行而遷移。 這會在兩個團隊間產生摩擦,故 此，我們建議安排人員進行協助遷移。

It’s also worth noting that even with the force of policy behind them, compulsory deprecations can still face political hurdles. Imagine trying to enforce a compulsory deprecation effort when the last remaining user of the old system is a critical piece of infrastructure your entire organization depends on. How willing would you be to break that infrastructure—and, transitively, everybody that depends on it—just for the sake of making an arbitrary deadline? It is hard to believe the deprecation is really compulsory if that team can veto its progress.

還值得注意的是，即使有策略支援，強制性“棄用”仍可能面臨政治障礙。 想象一下，當舊系統的最後一個剩餘使用者 是整個組織所依賴的關鍵基礎架構時， 你會願意為了在截止日期前完成遷移而破壞那個基礎設施及所有依賴它的 系統嗎？ 如果該團隊可以否決其進展，那它的強制性就值得懷疑。

Google’s monolithic repository and dependency graph gives us tremendous insight into how systems are used across our ecosystem. Even so, some teams might not even know they have a dependency on an obsolete system, and it can be difficult to discover these dependencies analytically. It’s also possible to find them dynamically through tests of increasing frequency and duration during which the old system is turned off temporarily. These intentional changes provide a mechanism for discovering unintended dependencies by seeing what breaks, thus alerting teams to a need to prepare for the upcoming deadline. Within Google, we occasionally change the name of implementation-only symbols to see which users are depending on them unaware.

Google 的中心程式碼儲存庫和依賴關係圖讓我們深入瞭解系統如何在我們的生態系統中使用。即便如此，一些團隊甚 至可能不知道他們依賴於一個過時的系統，並且很難透過分析發現這些依賴關係。也可以透過增加頻率和持續時間 的測試來動態找到它們，在此期間舊系統暫時關閉。這些有意的更改提供了一種機制，透過檢視中斷的內容來發現 意外的依賴關係，從而提醒團隊需要為即將到來的截止日期做好準備。在 Google 內部，我們偶爾會更改僅變數的 名稱，來檢視哪些使用者不知道依賴了它們。

Frequently at Google, when a system is slated for deprecation and removal, the team will announce planned outages of increasing duration in the months and weeks prior to the turndown. Similar to Google’s Disaster Recovery Testing (DiRT) exercises, these events often discover unknown dependencies between running systems. This incremental approach allows those dependent teams to discover and then plan for the system’s eventual removal, or even work with the deprecating team to adjust their timeline. (The same principles also apply for static code dependencies, but the semantic information provided by static analysis tools is often sufficient to detect all the dependencies of the obsolete system.)

在谷歌，當系統計劃“棄用”時，團隊經常會在關閉前的幾個月和幾周內宣佈計劃中斷，持續時間會增加。與 Google 的災難恢復測試 (DiRT) 類似，這些事件通常會發現正在執行的系統之間的未知依賴關係。這種漸進式方法允許那 些依賴的團隊發現依賴，然後為系統的最終移除做計劃，甚至與“棄用”團隊合作調整他們的時間表。 （同樣的原則 也適用於靜態程式碼依賴，但靜態分析工具提供的語義資訊通常足以檢測過時系統的所有依賴。）


### （棄用警告）Deprecation Warnings

For both advisory and compulsory deprecations, it is often useful to have a programmatic way of marking systems as deprecated so that users are warned about their use and encouraged to move away. It’s often tempting to just mark something as deprecated and hope its uses eventually disappear, but remember: “hope is not a strategy.” Deprecation warnings can help prevent new uses, but rarely lead to migration of existing systems.

對於建議性和強制“棄用”，以程式化的方式將系統標記為“棄用”通常很有用，這樣使用者就會及時的發現警告並遠離它。 將某些東西標記為已“棄用”並希望它的使用最終消失通常很誘人，但請記住：“希望不是一種策略。” “棄用”警告可以 減少它的新增使用者，但很少導致現有系統的遷移。

What usually happens in practice is that these warnings accumulate over time. If they are used in a transitive context (for example, library A depends on library B, which depends on library C, and C issues a warning, which shows up when A is built), these warnings can soon overwhelm users of a system to the point where they ignore them altogether. In health care, this phenomenon is known as “alert fatigue.”

在實踐中通常會發生這些警告隨著時間的推移而累積。 如果它們在傳遞上下文中使用（例如，函式庫 A 依賴於函式庫 B， 而函式庫 B 又依賴於函式庫 C，而 C 發出警告，並在建構 A 時顯示），則這些警告很快就會使系統使用者不知所措 他們完 全忽略它們的點。 在醫療保健領域，這種現象被稱為“警覺疲勞”。

Any deprecation warning issued to a user needs to have two properties: actionability and relevance. A warning is actionable if the user can use the warning to actually perform some relevant action, not just in theory, but in practical terms, given the expertise in that problem area that we expect for an average engineer. For example, a tool might warn that a call to a given function should be replaced with a call to its updated counterpart, or an email might outline the steps required to move data from an old system to a new one. In each case, the warning provided the next steps that an engineer can perform to no longer depend on the deprecated system.2

向用戶發出的任何“棄用”警告都需要具有兩個屬性：可操作性和相關性。 如果使用者可以使用警告來實際執行某些相 關操作，則警告是可操作的，不僅在理論上，而且在實踐中，即要提供可操作的遷移步驟，而不僅僅是一個警告。

A warning can be actionable, but still be annoying. To be useful, a deprecation warning should also be relevant. A warning is relevant if it surfaces at a time when a user actually performs the indicated action. Warning about the use of a deprecated function is best done while the engineer is writing code that uses that function, not after it has been checked into the repository for several weeks. Likewise, an email for data migration is best sent several months before the old system is removed rather than as an afterthought a weekend before the removal occurs.

警告可能是可行的，但仍然很煩人。 為了有用，“棄用”警告也應該是相關的。 如果警告在使用者實際執行指示的操作 時出現，則該警告是相關的。 關於使用已“棄用”函式的警告最好在工程師編寫使用該函式的程式碼時完成，而不是在 將其簽入儲存函式庫數週後。 同樣，最好在刪除舊系統前幾個月傳送資料遷移電子郵件，而不是在刪除前的一個週末 之後才傳送。

It’s important to resist the urge to put deprecation warnings on everything possible. Warnings themselves are not bad, but naive tooling often produces a quantity of warning messages that can overwhelm the unsuspecting engineer. Within Google, we are very liberal with marking old functions as deprecated but leverage tooling such as ErrorProne or clang-tidy to ensure that warnings are surfaced in targeted ways. As discussed in Chapter 20, we limit these warnings to newly changed lines as a way to warn people about new uses of the deprecated symbol. Much more intrusive warnings, such as for deprecated targets in the dependency graph, are added only for compulsory deprecations, and the team is actively moving users away. In either case, tooling plays an important role in surfacing the appropriate information to the appropriate people at the proper time, allowing more warnings to be added without fatiguing the user.

警告不是越多越好。 警告本身並不壞，但不成熟的工具通常會產生大量警告訊息，這些訊息可能會讓工程師不知 所措。 在 Google 內部，我們會將舊功能標記為已“棄用”，但會利用 ErrorProne 或 clang-tidy 等工具來確保以 有針對性的方式顯示警告。 正如第 20 章中所討論的，我們將這些警告限制在新更改的行中，以警告人們有關已 “棄用”符號的新用法。 更具侵入性的警告，例如依賴圖中已“棄用”的警告，僅針對強制“棄用”新增，並且團隊正在積極 地將使用者移走。 在任何一種情況下，工具都在適當的時間向適當的人提供適當的資訊方面發揮著重要作用，允許 新增更多警告而不會使使用者感到疲倦。

> 2 See https://abseil.io/docs/cpp/tools/api-upgrades for an example./
> 2 查閱https://abseil.io/docs/cpp/tools/api-upgrades 例子。


## 管理“棄用”的流程 (Managing the Deprecation Process)

Although they can feel like different kinds of projects because we’re deconstructing a system rather than building it, deprecation projects are similar to other software engineering projects in the way they are managed and run. We won’t spend too much effort going over similarities between those management efforts, but it’s worth pointing out the ways in which they differ.

“棄用”專案儘管與上線一個專案給你的感官不同，但它們的管理和執行方式卻是類似的。 我們不會花太多精力去討 論他們有何共同點，但有必要指出他們有何不同。


### 確定“棄用”的負責人 (Process Owners)

We’ve learned at Google that without explicit owners, a deprecation process is unlikely to make meaningful progress, no matter how many warnings and alerts a system might generate. Having explicit project owners who are tasked with managing and running the deprecation process might seem like a poor use of resources, but the alternatives are even worse: don’t ever deprecate anything, or delegate deprecation efforts to the users of the system. The second case becomes simply an advisory deprecation, which will never organically finish, and the first is a commitment to maintain every old system ad infinitum. Centralizing deprecation efforts helps better assure that expertise actually reduces costs by making them more transparent.

我們在 Google 瞭解到，如果沒有明確的Owner，無論系統產生了多少警報,“棄用”過程恐怕都不會太樂觀。 為了棄 用專門指定一個負責人似乎是對資源的浪費，永不“棄用”，或將“棄用”工作完全交給系統的使用者,恐怕會是一個更糟 的方案。 交給使用者來執行的方案，最多只能應對建議性“棄用”，恐怕它很難做到徹底地“棄用”，而永不“棄用”則相當 於無限期地維護著舊系統。 集中性的執行“棄用”則更專業更透明，從而真正達到降低成本的目的。

Abandoned projects often present a problem when establishing ownership and aligning incentives. Every organization of reasonable size has projects that are still actively used but that nobody clearly owns or maintains, and Google is no exception. Projects sometimes enter this state because they are deprecated: the original owners have moved on to a successor project, leaving the obsolete one chugging along in the basement, still a dependency of a critical project, and hoping it just fades away eventually.

廢棄的專案通常會在確定歸屬權上存在扯皮的情形。 每個小組都存在大量仍在使用卻無明確維護人的專案，谷歌 也不例外。 當一個專案存在這種情形時，通常說明它已被拋棄：即原維護人已參與到新專案開發維護中，老專案 則被棄之不顧，但卻仍然被某些關鍵專案所依賴，只寄希望於它慢慢消失在眾人視線中。

Such projects are unlikely to fade away on their own. In spite of our best hopes, we’ve found that these projects still require deprecation experts to remove them and prevent their failure at inopportune times. These teams should have removal as their primary goal, not just a side project of some other work. In the case of competing priorities, deprecation work will almost always be perceived as having a lower priority and rarely receive the attention it needs. These sorts of important-not-urgent cleanup tasks are a great use of 20% time and provide engineers exposure to other parts of the codebase.

但此類別專案不太可能自行消失。 儘管我們對之抱有最大的期寄，但我們發現,“棄用”這些專案仍然需要專人負責,否 則恐怕會造成意外的損失。 負責人應該將廢棄他們作為主要目標。 在排優先順序時，“棄用”通常會有較低的優先順序, 且少有人關注。 但實際上，這些重要但不緊急的清理工作,佔用掉程式設計師20%的工作時間,應該是個合適的數字。



### 制定里程碑 (Milestones)

When building a new system, project milestones are generally pretty clear: “Launch the frobnazzer features by next quarter.” Following incremental development practices, teams build and deliver functionality incrementally to users, who get a win whenever they take advantage of a new feature. The end goal might be to launch the entire system, but incremental milestones help give the team a sense of progress and ensure they don’t need to wait until the end of the process to generate value for the organization.

在建構新系統時，專案里程碑通常非常明確：如“在下個季度推出某項功能。” 。 遵循迭代式開發流程的團隊,通 常以積小成大的方式建構系統,並最終交付給使用者，只要他們使用了新功能，他們的目的便算得到了。 最終目標當 然是啟用整個系統，但增量迭代式的開發，則能讓團隊成員更有成就感，因他們無需等到專案結束就可體驗專案。

In contrast, it can often feel that the only milestone of a deprecation process is removing the obsolete system entirely. The team can feel they haven’t made any progress until they’ve turned out the lights and gone home. Although this might be the most meaningful step for the team, if it has done its job correctly, it’s often the least noticed by anyone external to the team, because by that point, the obsolete system no longer has any users. Deprecation project managers should resist the temptation to make this the only measurable milestone, particularly given that it might not even happen in all deprecation projects.

相反，對於“棄用”，它常會給人一種只有一個里程碑的錯覺，即完全乾掉老舊的專案。 下班時，團隊成員通常會有 沒取得任何進展的感覺。 幹掉一個老舊的專案對團隊成員來說雖是頗有意義，但對團隊之外的人來說卻是完全無感， 因老舊的系統已不再被任何服務呼叫。 故專案經理不應將完全根除舊專案當作唯一的里程碑。

Similar to building a new system, managing a team working on deprecation should involve concrete incremental milestones, which are measurable and deliver value to users. The metrics used to evaluate the progress of the deprecation will be different, but it is still good for morale to celebrate incremental achievements in the deprecation process. We have found it useful to recognize appropriate incremental milestones, such as deleting a key subcomponent, just as we’d recognize accomplishments in building a new product.

與新建專案一樣，“棄用”一個專案也該漸進的設定多個可量化的里程碑,用於評估“棄用”進度的指標會有差異，但階段 性的慶祝有助提升士氣。


### 工具加持 (Deprecation Tooling)

Much of the tooling used to manage the deprecation process is discussed in depth elsewhere in this book, such as the large-scale change (LSC) process (Chapter 22) or our code review tools (Chapter 19). Rather than talk about the specifics of the tools, we’ll briefly outline how those tools are useful when managing the deprecation of an obsolete system. These tools can be categorized as discovery, migration, and backsliding prevention tooling.

許多用於管理“棄用”過程的工具在本書的其他地方進行了深入討論，例如大規模變更 (LSC) 過程（第 22 章）或我 們的程式碼審查工具（第 19 章）。 我們不討論這些工具的細節，而是簡要概述如何讓這些工具在管理廢棄系統的 “棄用”時發輝作用。 這些工具可以歸類別為發現、遷移和倒回滾預防工具。


#### 發現使用者 (Discovery)

During the early stages of a deprecation process, and in fact during the entire process, it is useful to know how and by whom an obsolete system is being used. Much of the initial work of deprecation is determining who is using the old system—and in which unanticipated ways. Depending on the kinds of use, this process may require revisiting the deprecation decision once new information is learned. We also use these tools throughout the deprecation process to understand how the effort is progressing.

在早期階段，實際上在整個過程中，確認誰在使用及怎樣使用我們的廢棄專案很有必要。 初始工作通常是用於確 認誰在用、以及以怎樣的方式使用。根據使用的方式不同，有可能會推翻我們“棄用”的推進流程。 我們還在整個棄 用過程中使用這些工具來了解工作進展情況。

Within Google, we use tools like Code Search (see Chapter 17) and Kythe (see Chapter 23) to statically determine which customers use a given library, and often to sample existing usage to see what sorts of behaviors customers are unexpectedly depending on. Because runtime dependencies generally require some static library or thin client use, this technique yields much of the information needed to start and run a deprecation process. Logging and runtime sampling in production help discover issues with dynamic dependencies.

在谷歌內部，我們使用程式碼搜尋（見第 17 章）和 Kythe（見第 23 章）等工具來靜態地確定哪些客戶使用給定的 函式庫，並經常對現有使用情況進行抽樣，以瞭解客戶的使用方式。 由於執行時依賴項通常需要使用一些靜態函式庫或瘦 客戶端，因此該技術能提供大部分決策資訊。 而生產中的日誌記錄和執行時取樣有助於發現動態依賴項的問題。

Finally, we treat our global test suite as an oracle to determine whether all references to an old symbol have been removed. As discussed in Chapter 11, tests are a mechanism of preventing unwanted behavioral changes to a system as the ecosystem evolves. Deprecation is a large part of that evolution, and customers are responsible for having sufficient testing to ensure that the removal of an obsolete system will not harm them.

最後，我們將整合測試套件視為預言機，以確定是否已刪除對舊變數、函式的所有參考。 正如第 11 章所討論的， 測試是一種防止系統隨著生態系統發展而發生不必要的行為變化的機制。 “棄用”是這種演變的重要組成部分，客戶 有責任進行足夠的測試，以確保刪除過時的系統不會對他們造成危害。


#### 遷移 (Migration)

Much of the work of doing deprecation efforts at Google is achieved by using the same set of code generation and review tooling we mentioned earlier. The LSC process and tooling are particularly useful in managing the large effort of actually updating the codebase to refer to new libraries or runtime services.

在 Google “棄用”的大部分工作是透過使用我們之前提到的同一組程式碼產生和審查工具來完成的,即LSC工具集。 它 在程式碼儲存庫在引入新函式庫或執行時服務時會很有用。


#### 避免“棄用”專案被重新啟用 (Preventing backsliding)

Finally, an often overlooked piece of deprecation infrastructure is tooling for preventing the addition of new uses of the very thing being actively removed. Even for advisory deprecations, it is useful to warn users to shy away from a deprecated system in favor of a new one when they are writing new code. Without backsliding prevention, deprecation can become a game of whack-a-mole in which users constantly add new uses of a system with which they are familiar (or find examples of elsewhere in the codebase), and the deprecation team constantly migrates these new uses. This process is both counterproductive and demoralizing.

最後，一個經常被忽視的問題是新增功能重新使用了廢棄的專案。 即使對於建議性“棄用”，警告使用者在編寫新程式碼 時避免使用已“棄用”的系統而支援新系統也是很有用的。 如果沒有後退預防機制，“棄用”可能會變成一場打地鼠遊戲。 按下葫蘆浮起瓢是很影響士氣的。

To prevent deprecation backsliding on a micro level, we use the Tricorder static analysis framework to notify users that they are adding calls into a deprecated system and give them feedback on the appropriate replacement. Owners of deprecated systems can add compiler annotations to deprecated symbols (such as the @deprecated Java annotation), and Tricorder surfaces new uses of these symbols at review time. These annotations give control over messaging to the teams that own the deprecated system, while at the same time automatically alerting the change author. In limited cases, the tooling also suggests a push-button fix to migrate to the suggested replacement.

為了防止使用廢棄專案，我們使用 Tricorder 靜態分析框架來通知使用者他們正在呼叫一個“棄用”的系統中，並提供 替代方案。 廢棄系統的維護者應該將不推薦使用的符號新增編譯器註釋（例如@deprecated Java 註釋），並且 Tricorder 在審查時會將其傳送給廢棄專案的維護者。同時自動提醒呼叫者。 在某些情況下，該工具還能一鍵以 替代方案進行修復。

On a macro level, we use visibility whitelists in our build system to ensure that new dependencies are not introduced to the deprecated system. Automated tooling periodically examines these whitelists and prunes them as dependent systems are migrated away from the obsolete system.

在巨集觀層面上，我們在建構系統中使用可見性白名單來確保不會將新的依賴項引入已“棄用”的系統。 自動化工具會 定期檢查這些白名單，並在依賴系統從過時系統遷移時對其進行刪剪。


## 結論 (Conclusion)

Deprecation can feel like the dirty work of cleaning up the street after the circus parade has just passed through town, yet these efforts improve the overall software ecosystem by reducing maintenance overhead and cognitive burden of engineers. Scalably maintaining complex software systems over time is more than just building and running software: we must also be able to remove systems that are obsolete or otherwise unused.

A complete deprecation process involves successfully managing social and technical challenges through policy and tooling. Deprecating in an organized and well- managed fashion is often overlooked as a source of benefit to an organization, but is essential for its long-term sustainability.

“棄用”感覺就像馬戲團剛剛穿過城鎮後，清理街道的骯髒工作，但它能透過減少維護開銷和工程師的認知負擔來改 善整個軟體生態系統。 隨著時間的推移，複雜系統的維護,不僅僅是包含建構和執行那麼簡單,還應包含清理過時 的老舊系統。

完整的“棄用”過程涉及到管理和技術兩個層面的挑戰。 有效地管理“棄用”通常因不會帶來盈利而被輕忽，但它對 其長期可持續性維護卻至關重要。


## TL;DRs  內容提要

-   Software systems have continuing maintenance costs that should be weighed against the costs of removing them.
-   Removing things is often more difficult than building them to begin with because existing users are often using the system beyond its original design.
-   Evolving a system in place is usually cheaper than replacing it with a new one, when turndown costs are included.
-   It is difficult to honestly evaluate the costs involved in deciding whether to deprecate: aside from the direct maintenance costs involved in keeping the old system around, there are ecosystem costs involved in having multiple similar systems to choose between and that might need to interoperate. The old system might implicitly be a drag on feature development for the new. These ecosystem costs are diffuse and difficult to measure. Deprecation and removal costs are often similarly diffuse.

-   軟體系統具有持續的維護成本，應與刪除它們的成本進行權衡。
-   刪除東西通常比開始建構它們更困難，因為現有使用者經常使用超出其原始設計意圖的系統。
-   如果將停機成本包括在內，就地改進系統通常比更換新系統便宜。
-   很難如實地評估 “棄用”所涉及的成本：除了保留舊系統所涉及的直接維護成本外，還有多個相似系統可供選擇 所涉及的生態成本，互有干涉。 舊系統可能會暗中拖累新系統的功能開發。 這些不同的生態所帶來的成本則分 散且難以衡量。 “棄用”成本通常同樣分散。
