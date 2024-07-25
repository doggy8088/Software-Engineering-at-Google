
**CHAPTER 21**

# Dependency Management

# 第二十一章 依賴管理

**Written by Titus Winters**

**Edited by Lisa Carey**

Dependency management—the management of networks of libraries, packages, and dependencies that we don」t control—is one of the least understood and most challenging problems in software engineering. Dependency management focuses on questions like: how do we update between versions of external dependencies? How do we describe versions, for that matter? What types of changes are allowed or expected in our dependencies? How do we decide when it is wise to depend on code produced by other organizations?

依賴管理——管理我們無法控制的函式庫、套件和依賴關係的網路——是軟體工程中最不為人理解和最有挑戰性的問題之一。依賴管理關注的問題包括：我們如何更新外部依賴的版本？為此，我們如何描述版本？在我們的依賴關係中，哪些型別的變化是允許的或預期的？我們如何決定何時依賴其他組織生產的程式碼是明智的？

For comparison, the most closely related topic here is source control. Both areas describe how we work with source code. Source control covers the easier part: where do we check things in? How do we get things into the build? After we accept the value of trunk-based development, most of the day-to-day source control questions for an organization are fairly mundane: 『I」ve got a new thing, what directory do I add it to?』

作為比較，與依賴管理最密切相關的主題是原始碼控制。這兩個領域都描述了我們如何處理原始碼。原始碼控制涵蓋了比較容易的部分：我們在哪裡檢查原始碼？我們如何將原始碼放入建立中？在我們接受了基於主幹的開發的價值之後，對於一個組織來說，大多數日常的原始碼控制問題都是相當平常的："我有一個新的東西，我應該把它新增到什麼目錄？"

Dependency management adds additional complexity in both time and scale. In a trunk-based source control problem, it」s fairly clear when you make a change that you need to run the tests and not break existing code. That」s predicated on the idea that you」re working in a shared codebase, have visibility into how things are being used, and can trigger the build and run the tests. Dependency management focuses on the problems that arise when changes are being made outside of your organization, without full access or visibility. Because your upstream dependencies can」t coordinate with your private code, they are more likely to break your build and cause your tests to fail. How do we manage that? Should we not take external dependencies? Should we ask for greater consistency between releases of external dependencies? When do we update to a new version?

依賴管理在時間和規模上都增加了額外的複雜性。在一個基於主幹的原始碼控制問題中，當你做一個改變時，你需要執行測試並且不破壞現有的程式碼，這是相當清楚的。這是基於這樣的想法：你在一個共享的程式碼庫中工作，能夠瞭解事物的使用方式，並且能夠觸發建立和執行測試的想法。依賴管理關注的是在你的組織之外進行改變時出現的問題，沒有完全的訪問許可權或可見性。因為你的上游依賴不能與你的私有程式碼協調，它們更有可能破壞你的建立，導致你的測試失敗。我們如何管理這個問題？我們不應該接受外部依賴嗎？我們是否應該要求外部依賴的版本之間更加一致？我們什麼時候更新到一個新的版本？

Scale makes all of these questions more complex, with the realization that we aren」t really talking about single dependency imports, and in the general case that we」re depending on an entire network of external dependencies. When we begin dealing with a network, it is easy to construct scenarios in which your organization」s use of two dependencies becomes unsatisfiable at some point in time. Generally, this happens because one dependency stops working without some requirement,[^1] whereas the other is incompatible with the same requirement. Simple solutions about how to manage a single outside dependency usually fail to account for the realities of managing a large network. We」ll spend much of this chapter discussing various forms of these conflicting requirement problems.

規模使所有這些問題變得更加複雜，因為我們意識到我們實際上並不是在討論單個依賴項匯入，而且在一般情況下，我們依賴於整個外部依賴網路。當我們開始處理網路時，很容易建立這樣的場景：你的組織對兩個依賴項的使用在某個時間點變得不能滿足。通常，這是因為一個依賴項在無法滿足某些要求停止工作，而另一個依賴項與相同的要求不相容。關於如何管理單個外部依賴關係的簡單解決方案通常沒有考慮到管理大型網路的現實情況。本章的大部分時間我們將討論這些相互衝突的需求問題的各種形式。

Source control and dependency management are related issues separated by the question: 『Does our organization control the development/update/management of this subproject?』 For example, if every team in your company has separate repositories, goals, and development practices, the interaction and management of code produced by those teams is going to have more to do with dependency management than source control. On the other hand, a large organization with a (virtual?) single repository (monorepo) can scale up significantly farther with source control policies—this is Google」s approach. Separate open source projects certainly count as separate organizations: interdependencies between unknown and not-necessarily-collaborating projects are a dependency management problem. Perhaps our strongest single piece of advice on this topic is this: *All else being equal, prefer source control problems over dependency-management problems.* If you have the option to redefine 『organization』 more broadly (your entire company rather than just one team), that」s very often a good trade-off. Source control problems are a lot easier to think about and a lot cheaper to deal with than dependency-management ones.

原始碼管理和依賴管理是由以下問題分開的相關問題：『我們的組織是否控制此子專案的開發/更新/管理？』例如，如果貴公司的每個團隊都有單獨的版本庫、目標和開發實踐，這些團隊產生的程式碼的互動和管理將更多地涉及依賴管理，而不是原始碼控制。另一方面，一個擁有（虛擬？）單個版本庫（monorepo）的大型組織可以透過原始碼控制策略進一步擴充套件，這是Google的方法。獨立的開源專案當然被視為獨立的組織：*未知專案和非必要協作專案之間的相互依賴關係是一個依賴管理問題*。也許我們在這個話題上最有力的建議是：在其他條件相同的情況下，我們更喜歡原始碼管理問題，而不是依賴管理問題。如果你可以選擇更廣泛地重新定義『組織』（你的整個公司而不僅僅是一個團隊），這通常是一個很好的權衡。原始碼管理問題比依賴管理問題更容易思考，處理成本也更低。

As the Open Source Software (OSS) model continues to grow and expand into new domains, and the dependency graph for many popular projects continues to expand over time, dependency management is perhaps becoming the most important problem in software engineering policy. We are no longer disconnected islands built on one or two layers outside an API. Modern software is built on towering pillars of dependencies; but just because we can build those pillars doesn」t mean we」ve yet figured out how to keep them standing and stable over time.

隨著開源軟體（OSS）模式的不斷發展和擴充套件到新的領域，以及許多流行專案的依賴關係隨著時間的推移不斷擴大，依賴管理也許正在成為軟體工程策略中最重要的問題。我們開發的軟體不再是建立在API之外的一層或兩層上的斷開連線的孤島。現代軟體建立在高聳的依賴性支柱之上；但僅僅因為我們能夠建造這些支柱，並不意味著我們已經弄清楚如何讓它們長期保持穩定。

In this chapter, we」ll look at the particular challenges of dependency management, explore solutions (common and novel) and their limitations, and look at the realities of working with dependencies, including how we」ve handled things in Google. It is important to preface all of this with an admission: we」ve invested a lot of *thought* into this problem and have extensive experience with refactoring and maintenance issues that show the practical shortcomings with existing approaches. We don」t have firsthand evidence of solutions that work well across organizations at scale. To some extent, this chapter is a summary of what we know does not work (or at least might not work at larger scales) and where we think there is the potential for better outcomes. We definitely cannot claim to have all the answers here; if we could, we wouldn」t be calling this one of the most important problems in software engineering.

在本章中，我們將介紹依賴管理的特殊挑戰，探索解決方案（常見的和新穎的）及其侷限性，並介紹使用依賴關係的現實情況，包括我們在 Google 中處理事情的方式。在所有這些之前，我們必須承認：我們在這個問題上投入了大量的精力，並且在重構和維護問題上擁有豐富的經驗這表明了現有方法的實際缺陷。我們沒有第一手證據表明解決方案能夠在大規模的組織中很好地工作。在某種程度上，本章總結了我們所知道的不起作用（或者至少在更大範圍內可能不起作用）以及我們認為有可能產生更好結果的地方。我們絕對不能聲稱這裡有所有的答案；如果可以，我們就不會把這稱為軟體工程中最重要的問題之一。

> [^1]: This could be any of language version, version of a lower-level library, hardware version, operating system, compiler flag, compiler version, and so on.
>
> 1 這可以是任何語言版本、較低級別函式庫的版本、硬體版本、作業系統、編譯器標誌、編譯器版本等。

## Why Is Dependency Management So Difficult?  為什麼依賴管理如此困難？

Even defining the dependency-management problem presents some unusual challenges. Many half-baked solutions in this space focus on a too-narrow problem formulation: 『How do we import a package that our locally developed code can depend upon?』 This is a necessary-but-not-sufficient formulation. The trick isn」t just finding a way to manage one dependency—the trick is how to manage a *network* of dependencies and their changes over time. Some subset of this network is directly necessary for your first-party code, some of it is only pulled in by transitive dependencies. Over a long enough period, all of the nodes in that dependency network will have new versions, and some of those updates will be important.[^2] How do we manage the resulting cascade of upgrades for the rest of the dependency network? Or, specifically, how do we make it easy to find mutually compatible versions of all of our dependencies given that we do not control those dependencies? How do we analyze our dependency network? How do we manage that network, especially in the face of an ever-growing graph of dependencies?

即使是定義依賴管理問題也會帶來一些不尋常的挑戰。這個領域的許多半生不熟的解決方案都集中在一個狹義的問題上。"我們如何匯入一個我們本地開發的程式碼可以依賴的套件？" 這是一個必要但並不充分的表述。訣竅不只是找到一種方法來管理一個依賴關係——訣竅是如何管理一個依賴關係的網路以及它們隨時間的變化。這個網路中的一些子集對於你的第一方程式碼來說是直接必要的，其中一些只是由傳遞依賴拉進來的。在一個足夠長的時期內，這個依賴網路中的所有節點都會有新的版本，其中一些更新會很重要。或者，具體來說，鑑於我們並不控制這些依賴關係，我們如何使其容易找到所有依賴關係的相互相容的版本？我們如何分析我們的依賴網路？我們如何管理這個網路，尤其是在面對不斷增長的依賴關係的時候？

### Conflicting Requirements and Diamond Dependencies  衝突的需求和菱形依賴

The central problem in dependency management highlights the importance of thinking in terms of dependency networks, not individual dependencies. Much of the difficulty stems from one problem: what happens when two nodes in the dependency network have conflicting requirements, and your organization depends on them both? This can arise for many reasons, ranging from platform considerations (operating system [OS], language version, compiler version, etc.) to the much more mundane issue of version incompatibility. The canonical example of version incompatibility as an unsatisfiable version requirement is the *diamond dependency* problem. Although we don」t generally include things like 『what version of the compiler』 are you using in a dependency graph, most of these conflicting requirements problems are isomorphic to 『add a (hidden) node to the dependency graph representing this requirement.』 As such, we」ll primarily discuss conflicting requirements in terms of diamond dependencies, but keep in mind that libbase might actually be absolutely any piece of software involved in the construction of two or more nodes in your dependency network.

依賴管理的核心問題強調從依賴關係網路而不是單個依賴關係角度思考的重要性。大部分困難源於一個問題：當依賴網路中的兩個節點有衝突的要求，而你的組織同時依賴它們時，會發生什麼？這可能有很多原因，從平台考慮（作業系統[OS]、語言版本、編譯器版本等）到更常見的版本不相容問題。作為一個不可滿足的版本要求，版本不相容的典型例子是*菱形依賴*問題。雖然我們通常不包括像 "你使用的是什麼版本的編譯器 "這樣的東西，但大多數這些衝突的需求問題都與 "在代表這個需求的依賴圖中新增一個（隱藏的）節點 "同構。因此，我們將主要討論菱形依賴關係方面的衝突需求，但請記住，libbase 實際上絕對可能是參與建立你的依賴關係網路中的兩個或多個節點的任何軟體。

The diamond dependency problem, and other forms of conflicting requirements, require at least three layers of dependency, as demonstrated in [Figure 21-1](#_bookmark1857).

菱形依賴問題，以及其他形式的衝突需求，需要至少三層依賴關係，如圖21-1所示。

![Figure 21-1](./images/Figure%2021-1.png)

*Figure* *21-1.* *The* *diamond* *dependency* *problem*  *菱形依賴問題*

In this simplified model, libbase is used by both liba and libb, and liba and libb are both used by a higher-level component libuser. If libbase ever introduces an incompatible change, there is a chance that liba and libb, as products of separate organizations, don」t update simultaneously. If liba depends on the new libbase version and libb depends on the old version, there」s no general way for libuser (aka your code) to put everything together. This diamond can form at any scale: in the entire network of your dependencies, if there is ever a low-level node that is required to be in two incompatible versions at the same time (by virtue of there being two paths from some higher level node to those two versions), there will be a problem.

在這個簡化模型中，liba和libb都使用libbase，而liba和libb都由更高階別的元件libuser使用。如果libbase引入了一個不相容的變更，那麼作為獨立組織的產品，liba和libb可能不會同時更新。如果liba依賴於新的libbase版本，而libb依賴於舊版本，那麼libuser（也就是你的程式碼）沒有通用的方法來組合所有內容。這個菱形依賴可以在任何規模形成：在依賴關係的整個網路中，如果有一個低階節點需要同時處於兩個不相容的版本中（由於從某個高階節點到這兩個版本有兩條路徑），那麼就會出現問題。

Different programming languages tolerate the diamond dependency problem to different degrees. For some languages, it is possible to embed multiple (isolated) versions of a dependency within a build: a call into libbase from liba might call a different version of the same API as a call into libbase from libb. For example, Java provides fairly well-established mechanisms to rename the symbols provided by such a dependency.[^3] Meanwhile, C++ has nearly zero tolerance for diamond dependencies in a normal build, and they are very likely to trigger arbitrary bugs and undefined behavior (UB) as a result of a clear violation of C++」s [One Definition Rule](https://oreil.ly/VTZe5). You can at best use a similar idea as Java」s shading to hide some symbols in a dynamic-link library (DLL) or in cases in which you」re building and linking separately. However, in all programming languages that we」re aware of, these workarounds are partial solutions at best: embedding multiple versions can be made to work by tweaking the names of *functions*, but if there are *types* that are passed around between dependencies, all bets are off. For example, there is simply no way for a map defined in libbase v1 to be passed through some libraries to an API provided by libbase v2 in a semantically consistent fashion. Language-specific hacks to hide or rename entities in separately compiled libraries can provide some cushion for diamond dependency problems, but are not a solution in the general case.

不同的程式語言對菱形依賴問題的容忍程度不同。對於某些語言來說，可以在建立過程中嵌入一個依賴關係的多個（孤立的）版本：從liba呼叫libbase可能與從libb呼叫libbase呼叫相同API的不同版本。例如，Java 提供了相當完善的機制來重新命名這種依賴關係所提供的符號。 同時，C++ 對正常建立中的菱形依賴關係的容忍度幾乎為零，由於明顯違反了 C++ 的 [One Definition Rule](https://oreil.ly/VTZe5) ，它們非常可能引發明顯 bug 和未定義行為（UB）。在動態連結庫（DLL）中或者在單獨建立和連結的情況下，你最多可以使用與Java的shading思想來隱藏一些符號。然而，在我們所知道的所有程式語言中，這些變通方法充其量只是部分解決方案：透過調整*函式*的名稱，可以使嵌入的多個版本發揮作用，但如果有*型別*在依賴關係之間傳遞，所有的下注都會無效。例如，libbase v1中定義的`map`型別根本不可能以語義一致的方式透過一些函式庫傳遞給libbase v2提供的API。在單獨編譯的庫中隱藏或重新命名實體的特定語言黑科技可以為菱形依賴問題提供一些緩衝，但在一般情況下並不是一個解決方案。

If you encounter a conflicting requirement problem, the only easy answer is to skip forward or backward in versions for those dependencies to find something compatible. When that isn」t possible, we must resort to locally patching the dependencies in question, which is particularly challenging because the cause of the incompatibility in both provider and consumer is probably not known to the engineer that first discovers the incompatibility. This is inherent: liba developers are still working in a compatible fashion with libbase v1, and libb devs have already upgraded to v2. Only a dev who is pulling in both of those projects has the chance to discover the issue, and it」s certainly not guaranteed that they are familiar enough with libbase and liba to work through the upgrade. The easier answer is to downgrade libbase and libb, although that is not an option if the upgrade was originally forced because of security issues.

如果你遇到一個衝突的需求問題，唯一簡單的答案是向前或向後跳過這些依賴的版本，以找到相容的版本。當這不可能時，我們必須求助於本地修補有問題的依賴關係，這特別具有挑戰性，因為首先發現不相容的工程師可能不知道提供者和使用者中不相容的原因。這是固有的：liba的開發者還在以相容的方式與libbase v1工作，而libb的開發者已經升級到了v2。只有同時參與這兩個專案的開發人員才有機會發現問題，當然也不能保證他們對libbase和liba足夠熟悉，能夠完成升級。更簡單的答案是降級libbase和libb，儘管如果升級最初是因為安全問題而被迫進行的，那麼這不是一個選項。

Systems of policy and technology for dependency management largely boil down to the question, 『How do we avoid conflicting requirements while still allowing change among noncoordinating groups?』 If you have a solution for the general form of the diamond dependency problem that allows for the reality of continuously changing requirements (both dependencies and platform requirements) at all levels of the network, you」ve described the interesting part of a dependency-management solution.

依賴管理的策略和技術體系在很大程度上歸結為一個問題："我們如何避免衝突的需求，同時仍然允許非協調組之間的變化？" 如果你有一個菱形依賴問題的一般形式的解決方案，允許在網路的各個層面不斷變化的需求（包括依賴和平台需求）的現實，你已經描述了依賴管理解決方案的有趣部分。

> [^2]: For instance, security bugs, deprecations, being in the dependency set of a higher-level dependency that has a security bug, and so on.
>
> 2 例如，安全缺陷、棄用、處於具有安全缺陷的更高階別依賴項的依賴項集中，等等。
>
> [^3]: This is called shading or versioning.
>
> 3 這稱為著色或版本控制。

## Importing Dependencies 匯入依賴

In programming terms, it」s clearly better to reuse some existing infrastructure rather than build it yourself. This is obvious, and part of the fundamental march of technology: if every novice had to reimplement their own JSON parser and regular expression engine, we」d never get anywhere. Reuse is healthy, especially compared to the cost of redeveloping quality software from scratch. So long as you aren」t downloading trojaned software, if your external dependency satisfies the requirements for your programming task, you should use it.

在編程方面，重用一些現有的基礎設施顯然比自己建立它更好。這是顯而易見的，也是技術發展的一部分：如果每個新手都必須重新實現他們自己的JSON語法分析器和正則表示式引擎，我們就永遠不會有任何進展。重用是健康的，特別是與從頭開始重新開發高質量軟體的成本相比。只要你下載的不是木馬軟體，如果你的外部依賴滿足了你的程式設計任務的要求，你就應該使用它。

### Compatibility Promises  承諾相容性

When we start considering time, the situation gains some complicated trade-offs. Just because you get to avoid a *development* cost doesn」t mean importing a dependency is the correct choice. In a software engineering organization that is aware of time and change, we need to also be mindful of its ongoing maintenance costs. Even if we import a dependency with no intent of upgrading it, discovered security vulnerabilities, changing platforms, and evolving dependency networks can conspire to force that upgrade, regardless of our intent. When that day comes, how expensive is it going to be? Some dependencies are more explicit than others about the expected maintenance cost for merely using that dependency: how much compatibility is assumed? How much evolution is assumed? How are changes handled? For how long are releases supported?

當我們開始考慮時間時，情況就會出現一些複雜的權衡。僅僅因為你可以避免*開發*的成本，並不意味著匯入一個依賴關係是正確的選擇。在一個瞭解時間和變化的軟體工程組織中，我們還需要注意其持續的維護成本。即使我們在匯入依賴關係時並不打算對其進行升級，被發現的安全漏洞、不斷變化的平台和不斷發展的依賴關係網路也會合力迫使我們進行升級，而不管我們的意圖如何。當這一天到來時，它將會有多高的成本？有些依賴關係比其他依賴關係更清楚地說明了僅僅使用該依賴關係的預期維護成本：假定有多少相容性？假設有多大的演變？如何處理變化？版本支援多長時間？

We suggest that a dependency provider should be clearer about the answers to these questions. Consider the example set by large infrastructure projects with millions of users and their compatibility promises.

我們建議，依賴提供者應該更清楚地瞭解這些問題的答案。考慮一下擁有數百萬使用者的大型基礎設施專案及其相容性承諾所樹立的榜樣。

#### C++

For the C++ standard library, the model is one of nearly indefinite backward compatibility. Binaries built against an older version of the standard library are expected to build and link with the newer standard: the standard provides not only API compatibility, but ongoing backward compatibility for the binary artifacts, known as *ABI compatibility*. The extent to which this has been upheld varies from platform to platform. For users of gcc on Linux, it」s likely that most code works fine over a range of roughly a decade. The standard doesn」t explicitly call out its commitment to ABI compatibility—there are no public-facing policy documents on that point. However, the standard does publish [Standing Document 8 ](https://oreil.ly/LoJq8)(SD-8), which calls out a small set of types of change that the standard library can make between versions, defining implicitly what type of changes to be prepared for. Java is similar: source is compatible between language versions, and JAR files from older releases will readily work with newer versions.

對於C++標準庫來說，這種模式是一種幾乎無限期的向後相容性的。根據標準庫的舊版本建立的二進位檔案有望與較新的標準進行建立和連結：標準不僅提供了API相容性，還為二進位工件提供了持續的向後相容性，即所謂的*ABI相容性*。這一支援程度在不同的平台是不同的。對於Linux上的gcc使用者來說，可能大多數程式碼在大約十年的範圍內都能正常工作。該標準沒有明確指出它對ABI相容性的承諾——在這一點上沒有面向公眾的政策檔案。然而，該標準確實發布了[常設檔案8](https://oreil.ly/LoJq8)(SD-8)，其中列出了標準庫在不同版本之間可以進行的一小部分變化型別，隱含地定義了需要準備的變化型別。Java也是如此：語言版本之間的原始碼是相容的，舊版本的JAR檔案很容易在新版本中執行。

#### Go

Not all languages prioritize the same amount of compatibility. The Go programming language explicitly promises source compatibility between most releases, but no binary compatibility. You cannot build a library in Go with one version of the language and link that library into a Go program built with a different version of the language.

並非所有的語言都優先考慮相同規格的相容性。Go程式語言明確承諾大多數版本之間的原始碼相容，但沒有二進位相容。你不能用一個版本的Go語言建立一個函式庫，然後把這個函式庫連結到用另一個版本的語言建立的Go程式中。

#### Abseil

Google」s Abseil project is much like Go, with an important caveat about time. We are unwilling to commit to compatibility *indefinitely*: Abseil lies at the foundation of most of our most computationally heavy services internally, which we believe are likely to be in use for many years to come. This means we」re careful to reserve the right to make changes, especially in implementation details and ABI, in order to allow better performance. We have experienced far too many instances of an API turning out to be confusing and error prone after the fact; publishing such known faults to tens of thousands of developers for the indefinite future feels wrong. Internally, we already have roughly 250 million lines of C++ code that depend on this library—we aren」t going to make API changes lightly, but it must be possible. To that end, Abseil explicitly does not promise ABI compatibility, but does promise a slightly limited form of API compatibility: we won」t make a breaking API change without also providing an automated refactoring tool that will transform code from the old API to the new transparently. We feel that shifts the risk of unexpected costs significantly in favor of users: no matter what version a dependency was written against, a user of that dependency and Abseil should be able to use the most current version. The highest cost should be 『run this tool,』 and presumably send the resulting patch for review in the mid-level dependency (liba or libb, continuing our example from earlier). In practice, the project is new enough that we haven」t had to make any significant API breaking changes. We can」t say how well this will work for the ecosystem as a whole, but in theory, it seems like a good balance for stability versus ease of upgrade.

谷歌的Abseil專案很像Go，對時間有一個重要的警告。我們不願意無限期地致力於相容性。Abseil是我們內部大多數計算量最大的服務的基礎，我們相信這些服務可能會在未來很多年內使用。這意味著我們小心翼翼地保留修改的權利，特別是在實現細節和ABI方面，以實現更好的效能。我們已經經歷了太多的例子，一個API在事後被證明是混亂和容易出錯的；在無限期的未來向成千上萬的開發者公佈這種已知的錯誤感覺是錯誤的。在內部，我們已經有大約2.5億行的C++程式碼依賴於這個函式庫，我們不會輕易改變API，但它必須是可以改變的。為此，Abseil明確地不承諾ABI的相容性，但確實承諾了一種稍微有限的API相容性：我們不會在不提供自動重構工具的情況下做出破壞性的API改變，該工具將透明地將程式碼從舊的API轉換到新的API。我們覺得這將意外成本的風險大大地轉移到了使用者身上：無論一個依賴關係是針對哪個版本編寫的，該依賴關係和Abseil的使用者都應該能夠使用最新的版本。最高的成本應該是 "執行這個工具"，並推測在中層級依賴關係（liba或libb，繼續我們前面的例子）中傳送產生的補丁以供審查。在實踐中，這個專案足夠新，我們沒有必要做任何重大的API破壞性改變。我們不能說這對整個生態系統會有多好的效果，但在理論上，這似乎是對穩定性和易升級的一個良好平衡。

#### Boost

By comparison, the Boost C++ library makes no promises of [compatibility between](https://www.boost.org/users/faq.html) [versions](https://www.boost.org/users/faq.html). Most code doesn」t change, of course, but 『many of the Boost libraries are actively maintained and improved, so backward compatibility with prior version isn」t always possible.』 Users are advised to upgrade only at a period in their project life cycle in which some change will not cause problems. The goal for Boost is fundamentally different than the standard library or Abseil: Boost is an experimental proving ground. A particular release from the Boost stream is probably perfectly stable and appropriate for use in many projects, but Boost」s project goals do not prioritize compatibility between versions—other long-lived projects might experience some friction keeping up to date. The Boost developers are every bit as expert as the developers for the standard library[^4]—none of this is about technical expertise: this is purely a matter of what a project does or does not promise and prioritize.

相比之下，Boost C++函式庫沒有承諾[不同版本](https://www.boost.org/users/faq.html)的相容性。當然，大多數程式碼不會改變，但 "許多Boost函式庫都在積極維護和改進，所以向後相容以前的版本並不總是可能的"。我們建議使用者只在專案生命週期的某個階段進行升級，因為在這個階段，一些變化不會造成問題。Boost的目標與標準庫或Abseil有根本的不同：Boost是一個實驗驗證場。Boost的某個版本可能非常穩定，適合在許多專案中使用，但是Boost的專案目標並不優先考慮版本之間的相容性——其他長期專案可能會遇到一些與最新版本保持同步的阻力。Boost的開發者和標準庫的開發者一樣都是專家——這與技術專長無關：這純粹是一個專案是否承諾和優先考慮的問題。

Looking at the libraries in this discussion, it」s important to recognize that these compatibility issues are *software engineering* issues, not *programming* issues. You can download something like Boost with no compatibility promise and embed it deeply in the most critical, long-lived systems in your organization; it will *work* just fine. All of the concerns here are about how those dependencies will change over time, keeping up with updates, and the difficulty of getting developers to worry about maintenance instead of just getting features working. Within Google, there is a constant stream of guidance directed to our engineers to help them consider this difference between 『I got it to work』 and 『this is working in a supported fashion.』 That」s unsurprising: it」s basic application of Hyrum」s Law, after all.

從這個討論中的函式庫來看，重要的是要認識到這些相容性問題是*軟體工程*問題，而不是*程式設計*問題。你可以下載像Boost這樣沒有相容性承諾的東西，並把它深深地嵌入到你的組織中最關鍵、生命週期最長的系統中；它可以*正常工作*。這裡所有的擔憂都是關於這些依賴關係會隨著時間的推移而改變，跟上更新的步伐，以及讓開發者擔心維護而不是讓功能正常工作的困難。在谷歌內部，有源源不斷的指導意見指向我們的工程師，幫助他們考慮『我讓它起作用了』和『這是以一種支援的方式起作用的』之間的區別。這並不奇怪：畢竟，這是海勒姆定律的基本應用。

Put more broadly: it is important to realize that dependency management has a wholly different nature in a programming task versus a software engineering task. If you」re in a problem space for which maintenance over time is relevant, dependency management is difficult. If you」re purely developing a solution for today with no need to ever update anything, it is perfectly reasonable to grab as many readily available dependencies as you like with no thought of how to use them responsibly or plan for upgrades. Getting your program to work today by violating everything in SD-8 and also relying on binary compatibility from Boost and Abseil works fine…so long as you never upgrade the standard library, Boost, or Abseil, and neither does anything that depends on you.

更廣泛地說：重要的是要意識到，依賴管理在編程任務和軟體工程任務中具有完全不同的性質。如果你所處的問題空間與隨時間的維護相關，則依賴關係管理很困難。如果你只是為今天開發一個解決方案，而不需要更新任何東西，那麼你完全可以隨心所欲地抓取許多現成的依賴關係，而不考慮如何負責任地使用它們或為升級做計劃。透過違反SD-8中的所有規定，並依靠Boost和Abseil的二進位相容性，使你的程式今天就能執行......只要你不升級標準庫、Boost或Abseil，也不升級任何依賴你的東西，就可以了。


> [^4]: In many cases, there is significant overlap in those populations.
>
> 4 在許多情況下，這些人群中存在著明顯的重疊。

### Considerations When Importing  匯入依賴的注意事項

Importing a dependency for use in a programming project is nearly free: assuming that you」ve taken the time to ensure that it does what you need and isn」t secretly a security hole, it is almost always cheaper to reuse than to reimplement functionality. Even if that dependency has taken the step of clarifying what compatibility promise it will make, so long as we aren」t ever upgrading, anything you build on top of that snapshot of your dependency is fine, no matter how many rules you violate in consuming that API. But when we move from programming to software engineering, those dependencies become subtly more expensive, and there are a host of hidden costs and questions that need to be answered. Hopefully, you consider these costs before importing, and, hopefully, you know when you」re working on a programming project versus working on a software engineering project.

匯入一個依賴項用於程式設計專案幾乎是免費的：假設你已經花了時間來確保它做了你需要的事情，並且沒有隱蔽的安全漏洞，那麼重用幾乎總是比重新實現功能要划算。即使該依賴項已經採取了澄清它將作出什麼相容性承諾的步驟，只要我們不曾升級，你在該依賴項的快照之上建立的任何東西都是好的，無論你在使用該API時違反了多少規則。但是，當我們從程式設計轉向軟體工程時，這些依賴關係的成本會變得微妙地更高，而且有一系列的隱藏成本和問題需要回答。希望你在匯入之前考慮到這些成本，而且，希望你知道你什麼時候是在做一個程式設計專案，而不是在做一個軟體工程專案。

When engineers at Google try to import dependencies, we encourage them to ask this (incomplete) list of questions first:

- Does the project have tests that you can run?
- Do those tests pass?
- Who is providing that dependency? Even among 『No warranty implied』 OSS projects, there is a significant range of experience and skill set—it」s a very different thing to depend on compatibility from the C++ standard library or Java」s Guava library than it is to select a random project from GitHub or npm. Reputation isn」t everything, but it is worth investigating.
- What sort of compatibility is the project aspiring to?
- Does the project detail what sort of usage is expected to be supported?
- How popular is the project?
- How long will we be depending on this project?
- How often does the project make breaking changes? 

當谷歌的工程師試圖匯入依賴時，我們鼓勵他們先問這個（不完整）的問題清單：

- 該專案是否有你可以執行的測試？
- 這些測試是否透過？
- 誰在提供這個依賴？即使在 "無擔保 "的開放原始碼軟體專案中，也有相當大的經驗和技能範圍——依賴C++標準庫或Java的Guava函式庫的相容性，與從GitHub或npm中隨機選擇一個專案是完全不同的事情。信譽不是一切，但它值得調研。
- 該專案希望達到什麼樣的相容性？
- 該專案是否詳細說明了預計會支援什麼樣的用法？
- 該專案有多受歡迎？
- 我們將在多長時間內依賴這個專案？
- 該專案多長時間做一次突破性的改變？專案多久進行一次突破性的變更？

Add to this a short selection of internally focused questions:

- How complicated would it be to implement that functionality within Google?
- What incentives will we have to keep this dependency up to date?
- Who will perform an upgrade?
- How difficult do we expect it to be to perform an upgrade?

在此基礎上，新增一些簡短的內部重點問題：
    
- 在谷歌內部實現該功能會有多複雜？
- 我們有什麼激勵措施來保持這個依賴的最新狀態？
- 誰來執行升級？
- 我們預計進行升級會有多大難度？


Our own Russ Cox has [written about this more extensively](https://research.swtch.com/deps). We can」t give a perfect formula for deciding when it」s cheaper in the long term to import versus reimplement; we fail at this ourselves, more often than not.

我們的Russ Cox已經[更廣泛地寫到了這一點]（https://research.swtch.com/deps）。我們無法給出一個完美的公式來決定從長遠來看，什麼時候引入和重新實施更划算；我們自己在這方面經常失敗。

### How Google Handles Importing Dependencies  Google如何處理匯入依賴

In short: we could do better.

簡言之：我們可以做得更好。

The overwhelming majority of dependencies in any given Google project are internally developed. This means that the vast majority of our internal dependency-management story isn」t really dependency management, it」s just source control—by design. As we have mentioned, it is a far easier thing to manage and control the complexities and risks involved in adding dependencies when the providers and consumers are part of the same organization and have proper visibility and Continuous Integration (CI; see Chapter 23) available. Most problems in dependency management stop being problems when you can see exactly how your code is being used and know exactly the impact of any given change. Source control (when you control the projects in question) is far easier than dependency management (when you don」t).

在任何特定的 Google 專案中，絕大多數的依賴都是內部開發的。這意味著，我們的內部依賴管理故事中的絕大部分並不是真正的依賴管理，它只是設計上的原始碼控制。正如我們所提到的，當提供者和消費者是同一組織的一部分，並且有適當的可見性和持續整合（CI；見第23章）時，管理和控制增加依賴關係所涉及的複雜性和風險是一件容易得多的事情。當你能準確地看到你的程式碼是如何被使用的，並準確地知道任何給定變化的影響時，依賴管理中的大多數問題就不再是問題了。原始碼控制（當你控制有關專案時）要比依賴管理（當你不控制時）容易得多。

That ease of use begins failing when it comes to our handling of external projects. For projects that we are importing from the OSS ecosystem or commercial partners, those dependencies are added into a separate directory of our monorepo, labeled *third_party*. Let」s examine how a new OSS project is added to *third_party*.

當涉及到我們對外部專案的處理時，這種易用性開始失效了。對於我們從開放原始碼軟體生態系統或商業夥伴那裡匯入的專案，這些依賴關係被新增到我們monorepo的一個單獨目錄中，標記為*third_party*。我們來看看一個新的OSS專案是如何被新增到*third_party*的。

Alice, a software engineer at Google, is working on a project and realizes that there is an open source solution available. She would really like to have this project completed and demo」ed soon, to get it out of the way before going on vacation. The choice then is whether to reimplement that functionality from scratch or download the OSS package and get it added to *third_party*. It」s very likely that Alice decides that the faster development solution makes sense: she downloads the package and follows a few steps in our *third_party* policies. This is a fairly simple checklist: make sure it builds with our build system, make sure there isn」t an existing version of that package, and make sure at least two engineers are signed up as OWNERS to maintain the package in the event that any maintenance is necessary. Alice gets her teammate Bob to say, 『Yes, I」ll help.』 Neither of them need to have any experience maintaining a *third_party* package, and they have conveniently avoided the need to understand anything about the *implementation* of this package. At most, they have gained a little experience with its interface as part of using it to solve the prevacation demo problem.

Alice是谷歌的一名軟體工程師，她正在做一個專案，並意識到有一個開源的解決方案可用。她真的很想盡快完成這個專案並進行示範，希望在去度假之前把它解決掉。然後的選擇是，是從頭開始重新實現這個功能，還是下載開放原始碼套件，並將其新增到*第三方*。很可能Alice決定更快的開發方案是有意義的：她下載了套件，並按照我們的*third_party*策略中的幾個步驟進行了操作。這是一個相當簡單的清單：確保它在我們的建立系統中建立，確保該軟體套件沒有現有的版本，並確保至少有兩名工程師註冊為OWNERS（所有者），在有必要進行任何維護時維護該軟體套件。Alice讓她的隊友Bob說，"是的，我會幫忙"。他們都不需要有維護*第三方*套件的經驗，而且他們很方便地避免了對這個套件的*實施*的瞭解。最多，他們對它的介面獲得了一點經驗，作為使用它來解決預先示範問題的一部分。

From this point on, the package is usually available to other Google teams to use in their own projects. The act of adding additional dependencies is completely transparent to Alice and Bob: they might be completely unaware that the package they downloaded and promised to maintain has become popular. Subtly, even if they are monitoring for new direct usage of their package, they might not necessarily notice growth in the *transitive* usage of their package. If they use it for a demo, while Charlie adds a dependency from within the guts of our Search infrastructure, the package will have suddenly moved from fairly innocuous to being in the critical infrastructure for important Google systems. However, we don」t have any particular signals surfaced to Charlie when he is considering whether to add this dependency.

從這時起，該軟體套件通常可以供其他谷歌團隊在他們自己的專案中使用。新增額外的依賴關係的行為對Alice和Bob來說是完全透明的：他們可能完全沒有意識到他們下載並承諾維護的軟體套件已經變得很流行。微妙的是，即使他們在監測他們的軟體套件的新的直接使用情況，他們也不一定會注意到他們的軟體套件的*間接*使用的增長。如果他們把它用於示範，而Charlie為我們的搜尋基礎設施的內部增加了一個依賴，那麼這個套件就會突然從相當無害的地方變成谷歌重要系統的關鍵基礎設施。然而，當Charlie考慮是否要新增這個依賴時，我們沒有任何特別的訊號提示給他。

Now, it」s possible that this scenario is perfectly fine. Perhaps that dependency is well written, has no security bugs, and isn」t depended upon by other OSS projects. It might be *possible* for it to go quite a few years without being updated. It」s not necessarily *wise* for that to happen: changes externally might have optimized it or added important new functionality, or cleaned up security holes before CVEs[^5] were discovered. The longer that the package exists, the more dependencies (direct and indirect) are likely to accrue. The more that the package remains stable, the more that we are likely to accrete Hyrum」s Law reliance on the particulars of the version that is checked into *third_party*.

現在，這種情況有可能是完美的。也許這個依賴項寫得很好，沒有安全漏洞，也沒有被其他OSS專案所依賴。這可能是*有可能的*，因為它可以在相當長的時間內不被更新。但這並不一定是明智之舉：外部的變化可能已經最佳化了它，或者增加了重要的新功能，或者在CVE被發現之前清理了安全漏洞。軟體套件存在的時間越長，依賴（直接和間接）就越多。軟體套件越是保持穩定，我們就越有可能增加海勒姆定律對被檢查到*第三方*的版本的特定依賴。

One day, Alice and Bob are informed that an upgrade is critical. It could be the disclosure of a security vulnerability in the package itself or in an OSS project that depends upon it that forces an upgrade. Bob has transitioned to management and hasn」t touched the codebase in a while. Alice has moved to another team since the demo and hasn」t used this package again. Nobody changed the OWNERS file. Thousands of projects depend on this indirectly—we can」t just delete it without breaking the build for Search and a dozen other big teams. Nobody has any experience with the implementation details of this package. Alice isn」t necessarily on a team that has a lot of experience undoing Hyrum」s Law subtleties that have accrued over time.

有一天，Alice和Bob被告知，升級是很關鍵的。這可能是軟體套件本身或依賴它的OSS專案中的安全漏洞被披露，從而迫使他們進行升級。Bob已經成為管理層，並且已經有一段時間沒有碰過程式碼庫了。Alice在示範後轉到了另一個團隊，並沒有再使用這個套件。沒有人改變OWNERS檔案。成千上萬的專案都間接地依賴於此--我們不能在不破壞Search和其他十幾個大團隊的建立的情況下直接刪除它。沒有人對這個套件的實現細節有任何經驗。Alice所在的團隊不一定有在消除海勒姆定律隨著時間積累的微妙之處方面經驗豐富。

All of which is to say: Alice and the other users of this package are in for a costly and difficult upgrade, with the security team exerting pressure to get this resolved immediately. Nobody in this scenario has practice in performing the upgrade, and the upgrade is extra difficult because it is covering many smaller releases covering the entire period between initial introduction of the package into *third_party* and the security disclosure.

所有這些都是說。Alice和這個軟體套件的其他使用者將面臨一次代價高昂而困難的升級，安全團隊正在施加壓力以立即解決這個問題。在這種情況下，沒有人有執行升級的經驗，而且升級是非常困難的，因為它涵蓋了許多較小的版本，涵蓋了從最初將軟體套件引入*第三方*到安全披露的整個時期。

Our *third_party* policies don」t work for these unfortunately common scenarios. We roughly understand that we need a higher bar for ownership, we need to make it easier (and more rewarding) to update regularly and more difficult for *third_party* packages to be orphaned and important at the same time. The difficulty is that it is difficult for codebase maintainers and *third_party* leads to say, 『No, you can」t use this thing that solves your development problem perfectly because we don」t have resources to update everyone with new versions constantly.』 Projects that are popular and have no compatibility promise (like Boost) are particularly risky: our developers might be very familiar with using that dependency to solve programming problems outside of Google, but allowing it to become ingrained into the fabric of our codebase is a big risk. Our codebase has an expected lifespan of decades at this point: upstream projects that are not explicitly prioritizing stability are a risk.

我們的*第三方套件*策略不適用於這些不幸的常見情況。我們大致明白，我們需要一個更高的所有權標準，我們需要讓定期更新更容易（和更多的回報），讓*第三方套件*更難成為孤島，同時也更重要。困難在於，程式碼庫維護者和*第三方套件*領導很難說："不，你不能使用這個能完美解決你的開發問題的東西，因為我們沒有資源不斷為大家更新新版本"。那些流行的、沒有相容性承諾的專案（比如Boost）尤其有風險：我們的開發者可能非常熟悉使用這種依賴關係來解決谷歌以外的程式設計問題，但允許它根植於我們的程式碼庫結構中是一個很大的風險。在這一點上，我們的程式碼庫有幾十年的預期壽命：上游專案如果沒有明確地優先考慮穩定性，就是一種風險。


> [^5]:	Common Vulnerabilities and Exposures.
>
> 5  常見漏洞和暴露

## Dependency Management, In Theory  理論上的依賴管理

Having looked at the ways that dependency management is difficult and how it can go wrong, let」s discuss more specifically the problems we」re trying to solve and how we might go about solving them. Throughout this chapter, we call back to the formulation, 『How do we manage code that comes from outside our organization (or that we don」t perfectly control): how do we update it, how do we manage the things it depends upon over time?』 We need to be clear that any good solution here avoids conflicting requirements of any form, including diamond dependency version conflicts, even in a dynamic ecosystem in which new dependencies or other requirements might be added (at any point in the network). We also need to be aware of the impact of time: all software has bugs, some of those will be security critical, and some fraction of our dependencies will therefore be *critical* to update over a long enough period of time.

在瞭解了依賴管理的困難以及它出錯的原因之後，讓我們更具體地討論我們要解決的問題以及我們如何去解決它們。在本章中，我們一直在呼籲："我們如何管理來自我們組織之外（或我們不能完全控制）的程式碼：我們如何更新它，如何管理它所依賴的東西？我們需要清楚，這裡的任何好的解決方案都會避免任何形式的需求衝突，包括菱形依賴版本衝突，甚至在一個動態的生態系統中，可能會增加新的依賴或其他需求（在網路中的任何一點）。我們還需要意識到時間的影響：所有的軟體都有bug，其中一些將是安全上的關鍵，因此我們的依賴中的一些部分將在足夠長的時間內可更新。

A stable dependency-management scheme must therefore be flexible with time and scale: we can」t assume indefinite stability of any particular node in the dependency graph, nor can we assume that no new dependencies are added (either in code we control or in code we depend upon). If a solution to dependency management prevents conflicting requirement problems among your dependencies, it」s a good solution. If it does so without assuming stability in dependency version or dependency fan-out, coordination or visibility between organizations, or significant compute resources, it」s a great solution.

因此，一個穩定的依賴管理方案必須在時間和規模上具有靈活性：我們不能假設依賴關係中任何特定節點的無限穩定，也不能假設沒有新的依賴被新增（無論是在我們控制的程式碼中還是在我們依賴的程式碼中）。如果一個依賴管理的解決方案能夠防止你的依賴關係中出現衝突的需求問題，那麼它就是一個好的解決方案。如果它不需要假設依賴版本或依賴扇出的穩定性，不需要組織間的協調或可見性，也不需要大量的計算資源，那麼它就是一個很好的解決方案。

When proposing solutions to dependency management, there are four common options that we know of that exhibit at least some of the appropriate properties: nothing ever changes, semantic versioning, bundle everything that you need (coordinating not per project, but per distribution), or Live at Head.

在提出依賴性管理的解決方案時，我們知道有四種常見的選擇，它們至少表現出一些適當的屬性：無任何更改、語義版本控制、捆綁你所需要的一切（不是按專案協調，而是按發行量協調），或直接使用最新版本。

### Nothing Changes (aka The Static Dependency Model)  沒有任何更改（也稱為靜態依賴模型）

The simplest way to ensure stable dependencies is to never change them: no API changes, no behavioral changes, nothing. Bug fixes are allowed only if no user code could be broken. This prioritizes compatibility and stability over all else. Clearly, such a scheme is not ideal due to the assumption of indefinite stability. If, somehow, we get to a world in which security issues and bug fixes are a nonissue and dependencies aren」t changing, the Nothing Changes model is very appealing: if we start with satisfiable constraints, we」ll be able to maintain that property indefinitely.

確保穩定的依賴關係的最簡單方法是永遠不要更改它們：不要改變API，不要改變行為，什麼都不要。只有在沒有用戶程式碼被破壞的情況下才允許修復錯誤。這將相容性和穩定性置於所有其他方面之上。顯然，這樣的方案並不理想，因為有無限期的穩定性的假設。如果以某種方式，我們遇到了一個安全問題和錯誤修復都不是問題，並且依賴關係不發生變化的世界，那麼 "無變化"模型就非常有吸引力：如果我們從可滿足的約束開始，我們就能無限期地保持這種屬性。

Although not sustainable in the long term, practically speaking, this is where every organization starts: up until you」ve demonstrated that the expected lifespan of your project is long enough that change becomes necessary, it」s really easy to live in a world where we assume that nothing changes. It」s also important to note: this is probably the right model for most new organizations. It is comparatively rare to know that you」re starting a project that is going to live for decades and have a *need* to be able to update dependencies smoothly. It」s much more reasonable to hope that stability is a real option and pretend that dependencies are perfectly stable for the first few years of a project.

雖然從長遠來看是不可持續的，但實際上，這是每個組織的出發點：直到你證明你的專案的預期生命週期足夠長，有必要進行更改，我們真的很容易生活在一個假設沒有變化的世界裡。同樣重要的是要注意：這可能是大多數新組織的正確模式。相對來說，很少有人知道你開始的專案將執行幾十年，並且*需要*能夠順利地更新依賴關係。希望穩定是一個真正的選擇，並假裝依賴關係在專案的前幾年是完全穩定的，這顯然要合理得多。

The downside to this model is that, over a long enough time period, it *is* false, and there isn」t a clear indication of exactly how long you can pretend that it is legitimate. We don」t have long-term early warning systems for security bugs or other critical issues that might force you to upgrade a dependency—and because of chains of dependencies, a single upgrade can in theory become a forced update to your entire dependency network.

這種模式的缺點是，在足夠長的時間內，它*是不存在*，並且沒有明確的跡象表明你可以假裝它是合理的。我們沒有針對安全漏洞或其他可能迫使你升級依賴關係的關鍵問題的長期預警系統，這些問題可能會迫使你升級一個依賴--由於依賴關係鏈的存在，有一個關係升級在理論上可以成為你整個依賴關係網路的強制更新。

In this model, version selection is simple: there are no decisions to be made, because there are no versions.

在這個模型中，版本選擇很簡單：因為沒有版本，所以不需要做出任何決定。

### Semantic Versioning  語義版本管理

The de facto standard for 『how do we manage a network of dependencies today?』 is semantic versioning (SemVer).[^6] SemVer is the nearly ubiquitous practice of representing a version number for some dependency (especially libraries) using three decimal-separated integers, such as 2.4.72 or 1.1.4. In the most common convention, the three component numbers represent major, minor, and patch versions, with the implication that a changed major number indicates a change to an existing API that can break existing usage, a changed minor number indicates purely added functionality that should not break existing usage, and a changed patch version is reserved for non-API-impacting implementation details and bug fixes that are viewed as particularly low risk.

"我們今天如何管理依賴關係網路？"事實上的標準是語義版本管理（SemVer）。SemVer是一種幾乎無處不在的做法，即用三個十進位分隔的整數來表示某些依賴關係（尤其是函式庫）的版本號，例如2.4.72或1.1.4。在最常見的慣例中，三個組成部分的數字代表主要、次要和補丁版本，其含義是：改變主要數字表示對現有API的改變，可能會破壞現有的使用，改變次要數字表示純粹增加的功能，不應該破壞現有的使用，而改變補丁版本是保留給非API影響的實施細節和被視為特別低風險的bug修復。

With the SemVer separation of major/minor/patch versions, the assumption is that a version requirement can generally be expressed as 『anything newer than,』 barring API-incompatible changes (major version changes). Commonly, we」ll see 『Requires libbase ≥ 1.5,』 that requirement would be compatible with any libbase in 1.5, including 1.5.1, and anything in 1.6 onward, but not libbase 1.4.9 (missing the API introduced in 1.5) or 2.x (some APIs in libbase were changed incompatibly). Major version changes are a significant incompatibility: because an existing piece of functionality has changed (or been removed), there are potential incompatibilities for all dependents. Version requirements exist (explicitly or implicitly) whenever one dependency uses another: we might see 『liba requires libbase ≥ 1.5』 and 『libb requires libbase ≥ 1.4.7.』

由於SemVer將主要/次要/補丁版本分離，假設版本需求通常可以表示為『任何更新的』，除非API不相容的更改（主版本更改）。通常，我們會看到 "Requires libbase ≥ 1.5"，這個需求會與1.5中的任何libbase相容，包括1.5.1，以及1.6以後的任何東西，但不包括libbase 1.4.9（缺少1.5中引入的API）或2.x（libbase中的一些API被不相容地更改）。主要的版本變化是一種重要的不相容：由於現有功能已更改（或已刪除），因此所有依賴項都存在潛在的不相容性。只要一個依賴關係使用另一個依賴關係，就會存在版本要求（明確地或隱含地）：我們可能看到 "liba requires libbase ≥ 1.5" 和 "libb requires libbase ≥ 1.4.7"。

If we formalize these requirements, we can conceptualize a dependency network as a collection of software components (nodes) and the requirements between them (edges). Edge labels in this network change as a function of the version of the source node, either as dependencies are added (or removed) or as the SemVer requirement is updated because of a change in the source node (requiring a newly added feature in a dependency, for instance). Because this whole network is changing asynchronously over time, the process of finding a mutually compatible set of dependencies that satisfy all the transitive requirements of your application can be challenging.[^7] Version- satisfiability solvers for SemVer are very much akin to SAT-solvers in logic and algorithms research: given a set of constraints (version requirements on dependency edges), can we find a set of versions for the nodes in question that satisfies all constraints? Most package management ecosystems are built on top of these sorts of graphs, governed by their SemVer SAT-solvers.

如果我們將這些要求標準化，我們可以將依賴網路概念化為軟體元件（節點）和它們之間的要求（邊緣）的集合。這個網路中的邊緣標籤作為源節點版本的函式而變化，要麼是由於依賴關係被新增（或刪除），要麼是由於源節點的變化而更新SemVer需求（例如，要求在依賴關係中新增新的功能）。由於整個依賴網路是隨著時間的推移而非同步變化的，因此，找到一組相互相容的依賴關係，以滿足應用程式的所有可傳遞需求的過程可能是一個具有挑戰性的過程。SemVer的版本滿足求解器非常類似於邏輯和演算法研究中的SAT求解器：給定一組約束（依賴邊的版本要求），我們能否為有關節點找到一組滿足所有約束的版本？大多數軟體套件管理生態系統都是建立在這類圖之上的，由其SemVer SAT求解器管理。

SemVer and its SAT-solvers aren」t in any way promising that there *exists* a solution to a given set of dependency constraints. Situations in which dependency constraints cannot be satisfied are created constantly, as we」ve already seen: if a lower-level component (libbase) makes a major-number bump, and some (but not all) of the libraries that depend on it (libb but not liba) have upgraded, we will encounter the diamond dependency issue.

SemVer和它的SAT求解器並不保證對一組給定的依賴性約束*存在*的解決方案。正如我們已經看到的，無法滿足依賴性約束的情況不斷出現：如果一個較低級別的元件（libbase）進行了重大的數字升級，而一些（但不是全部）依賴它的函式庫（libb但不是liba）已經升級，我們就會遇到菱形依賴問題。

SemVer solutions to dependency management are usually SAT-solver based. Version selection is a matter of running some algorithm to find an assignment of versions for dependencies in the network that satisfies all of the version-requirement constraints. When no such satisfying assignment of versions exists, we colloquially call it 『dependency hell.』

SemVer對依賴管理的解決方案通常是基於SAT求解器的。版本選擇是一個執行某種演算法的問題，為依賴網路中的依賴關係找到一個滿足所有版本要求約束的版本分配。當不存在這種滿意的版本分配時，我們通俗稱它為 "依賴地獄"。

We」ll look at some of the limitations of SemVer in more detail later in this chapter.

我們將在本章後面詳細介紹SemVer的一些限制。

> [^6]: Strictly speaking, SemVer refers only to the emerging practice of applying semantics to major/minor/patch version numbers, not the application of compatible version requirements among dependencies numbered in that fashion. There are numerous minor variations on those requirements among different ecosystems, but in general, the version-number-plus-constraints system described here as SemVer is representative of the practice at large.
>
> 6 嚴格來說，SemVer只是指對主要/次要/補丁版本號應用語義的新興做法，而不是在以這種方式編號的依賴關係中應用相容的版本要求。在不同的生態系統中，這些要求有許多細微的變化，但總的來說，這裡描述的SemVer的版本號加約束系統是對整個實踐的代表。
>
> [^7]:  In fact, it has been proven that SemVer constraints applied to a dependency network are NP-complete.
>
> 7 事實上，已經證明SemVer約束應用於依賴網路是NP-C(NP-完備)。

### Bundled Distribution Models  捆綁分銷模式

As an industry, we」ve seen the application of a powerful model of managing dependencies for decades now: an organization gathers up a collection of dependencies, finds a mutually compatible set of those, and releases the collection as a single unit. This is what happens, for instance, with Linux distributions—there」s no guarantee that the various pieces that are included in a distro are cut from the same point in time. In fact, it」s somewhat more likely that the lower-level dependencies are somewhat older than the higher-level ones, just to account for the time it takes to integrate them.

作為一個行業，幾十年來我們已經看到了一個強大的依賴管理模型的應用：一個組織收集一組依賴項，找到一組相互相容的依賴項，並將這些依賴項作為一個單元發布。例如，這就是發生在Linux發行版上的情況——不能保證包含在發行版中的各個部分是在同一時間點上劃分的。事實上，更有可能的是，低階別的依賴關係比高階別的依賴關係要老一些，只是為了考慮到整合它們所需要的時間。

This 『draw a bigger box around it all and release that collection』 model introduces entirely new actors: the distributors. Although the maintainers of all of the individual dependencies may have little or no knowledge of the other dependencies, these higher-level *distributors* are involved in the process of finding, patching, and testing a mutually compatible set of versions to include. Distributors are the engineers responsible for proposing a set of versions to bundle together, testing those to find bugs in that dependency tree, and resolving any issues.

這一『圍繞這一切畫一個更大的盒子併發布該系列』的模式引入了全新的參與者：分銷商。儘管所有獨立依賴的維護者可能對其他依賴知之甚少或一無所知，但這些高層次分發者參與了查詢、修補和測試要包含的相互相容的版本集的過程。分銷商是工程師，負責提出一組捆綁在一起的版本，測試這些版本以發現依賴關係樹中的錯誤，並解決任何問題。

For an outside user, this works great, so long as you can properly rely on only one of these bundled distributions. This is effectively the same as changing a dependency network into a single aggregated dependency and giving that a version number. Rather than saying, 『I depend on these 72 libraries at these versions,』 this is, 『I depend on RedHat version N,』 or, 『I depend on the pieces in the NPM graph at time T.』

對於外部使用者來說，這非常有效，只要你能正確地依賴這些捆綁的發行版中的一個。這實際上等於把一個依賴網路變成單個聚合依賴關係併為其提供版本號相同。與其說 "我在這些版本中依賴於這72個函式庫"，不如說 "我依賴RedHat的版本N"，或者 "我依賴於時間T時NPM圖中的片段"。

In the bundled distribution approach, version selection is handled by dedicated distributors.

在捆綁式分銷方式中，版本選擇由專門的分銷商處理。

### Live at Head  直接使用最新版本

The model that some of us at Google[^8] have been pushing for is theoretically sound, but places new and costly burdens on participants in a dependency network. It」s wholly unlike the models that exist in OSS ecosystems today, and it is not clear how to get from here to there as an industry. Within the boundaries of an organization like Google, it is costly but effective, and we feel that it places most of the costs and incentives into the correct places. We call this model 『Live at Head.』 It is viewable as the dependency-management extension of trunk-based development: where trunk- based development talks about source control policies, we」re extending that model to apply to upstream dependencies as well.

我們谷歌的一些人一直在推動的模式在理論上是合理的，但給依賴網路的參與者帶來了新的、沉重的負擔。它完全不同於今天存在於開放原始碼軟體生態系統中的模式，而且不清楚作為一個行業如何從這裡走到那裡。在像谷歌這樣的組織的範圍內，它的成本很高，但很有效，我們覺得它把大部分的成本和激勵放到了正確的地方。我們稱這種模式為 "Live at Head"。它可以被看作是基於主幹的開發的依賴管理的延伸：基於主幹的開發討論原始碼控制策略時，我們將該模型擴充套件到應用於上游依賴關係。

Live at Head presupposes that we can unpin dependencies, drop SemVer, and rely on dependency providers to test changes against the entire ecosystem before committing. Live at Head is an explicit attempt to take time and choice out of the issue of dependency management: always depend on the current version of everything, and never change anything in a way in which it would be difficult for your dependents to adapt. A change that (unintentionally) alters API or behavior will in general be caught by CI on downstream dependencies, and thus should not be committed. For cases in which such a change *must* happen (i.e., for security reasons), such a break should be made only after either the downstream dependencies are updated or an automated tool is provided to perform the update in place. (This tooling is essential for closed- source downstream consumers: the goal is to allow any user the ability to update use of a changing API without expert knowledge of the use or the API. That property significantly mitigates the 『mostly bystanders』 costs of breaking changes.) This philosophical shift in responsibility in the open source ecosystem is difficult to motivate initially: putting the burden on an API provider to test against and change all of its downstream customers is a significant revision to the responsibilities of an API provider.

『Live at Head』的前提是我們可以解除依賴關係，放棄SemVer，並依靠依賴提供者在送出之前對整個生態系統進行測試。Live at Head是一個明確的嘗試，將時間和選擇權從依賴管理的問題中剝離出來：始終依賴所有內容的當前版本，遠不要以你的依賴關係難以適應的方式更改任何事情。一個（無意的）改變API或行為的變化，一般來說會被下游依賴的CI所捕獲，因此不應該送出。對於這種變化必須發生的情況（即出於安全原因），只有在更新了下游依賴關係或提供了自動化工具來執行更新後，才能進行這種中斷。(這種工具對於封閉原始碼的下游消費者來說是必不可少的：目標是允許任何使用者有能力更新對變化中的API的使用，而不需要對使用或API的專家知識。這一屬性大大減輕了破壞性變化的 "大部分旁觀者 "的成本）。在開源生態系統中，這種責任的哲學轉變最初是很難激勵的：把測試和改變所有下游客戶的負擔放在API提供者身上，是對API提供者責任的重大修改。

Changes in a Live at Head model are not reduced to a SemVer 『I think this is safe or not.』 Instead, tests and CI systems are used to test against visible dependents to determine experimentally how safe a change is. So, for a change that alters only efficiency or implementation details, all of the visible affected tests might likely pass, which demonstrates that there are no obvious ways for that change to impact users—it」s safe to commit. A change that modifies more obviously observable parts of an API (syntactically or semantically) will often yield hundreds or even thousands of test failures. It」s then up to the author of that proposed change to determine whether the work involved to resolve those failures is worth the resulting value of committing the change. Done well, that author will work with all of their dependents to resolve the test failures ahead of time (i.e., unwinding brittle assumptions in the tests) and might potentially create a tool to perform as much of the necessary refactoring as possible.

Live at Head模型中的變化不會被簡化為SemVer "我認為這很安全或不安全"。相反，測試和CI系統用於針對可見的依賴進行測試，以透過實驗確定變化的安全性。因此，對於一個只改變效率或實現細節的變化，所有可見的受影響的測試都可能透過，這表明該變化沒有明顯的影響使用者的方式——它是安全的送出。修改API中更明顯的可觀察部分（語法上或語義上），往往會產生成百上千的測試失敗。這時就需要修改建議的作者來決定解決這些故障的工作是否值得送出修改的結果。如果做得好，作者將與他們所有的依賴者一起工作，提前解決測試失敗的問題（即解除測試中的脆性假設），並有可能建立一個工具來執行儘可能多的必要重構。

The incentive structures and technological assumptions here are materially different than other scenarios: we assume that there exist unit tests and CI, we assume that API providers will be bound by whether downstream dependencies will be broken, and we assume that API consumers are keeping their tests passing and relying on their dependency in supported ways. This works significantly better in an open source ecosystem (in which fixes can be distributed ahead of time) than it does in the face of hidden/closed-source dependencies. API providers are incentivized when making changes to do so in a way that can be smoothly migrated to. API consumers are incentivized to keep their tests working so as not to be labeled as a low-signal test and potentially skipped, reducing the protection provided by that test.

這裡的激勵結構和技術假設與其他場景有實質性的不同：我們假設存在單元測試和CI，我們假設API提供者將受到下游依賴關係是否會被破壞的約束，我們假設API使用者保持他們的測試透過並以支援的方式依賴他們的依賴關係。這在一個開源的生態系統中（可以提前發布修復程式）比在面對隱藏/閉源的依賴關係時效果要好得多。API提供者在以一種可以順利遷移的方式進行更改時，會受到激勵。API使用者被激勵保持他們的測試工作，以避免被標記為低訊號測試並可能被跳過，從而減少該測試所提供的保護。

In the Live at Head approach, version selection is handled by asking 『What is the most recent stable version of everything?』 If providers have made changes responsibly, it will all work together smoothly.

在Live at Head方法中，透過詢問『哪個是最新的穩定版本？』來處理版本選擇。如果提供者能夠負責任地做出更改，則所有更改都將順利進行。

> [^8]: Especially the author and others in the Google C++ community.
>
> 8 特別是作者和其他在谷歌C++社群。


## The Limitations of SemVer  SemVer （語義版本管理）的侷限性

The Live at Head approach may build on recognized practices for version control (trunk-based development) but is largely unproven at scale. SemVer is the de facto standard for dependency management today, but as we」ve suggested, it is not without its limitations. Because it is such a popular approach, it is worth looking at it in more detail and highlighting what we believe to be its potential pitfalls.

Live at Head方法可能建立在公認的版本控制實踐（基於主幹的開發）的基礎之上，但在規模上基本沒有得到驗證。SemVer是當今依賴性管理的事實標準，但正如我們所建議的，它並非沒有侷限性。因為這是一種非常流行的方法，所以值得更詳細地研究它，並強調我們認為可能存在的陷阱。

There」s a lot to unpack in the SemVer definition of what a dotted-triple version number really means. Is this a promise? Or is the version number chosen for a release an estimate? That is, when the maintainers of libbase cut a new release and choose whether this is a major, minor, or patch release, what are they saying? Is it provable that an upgrade from 1.1.4 to 1.2.0 is safe and easy, because there were only API additions and bug fixes? Of course not. There」s a host of things that ill-behaved users of libbase could have done that could cause build breaks or behavioral changes in the face of a 『simple』 API addition.[^9] Fundamentally, you can」t *prove* anything about compatibility when only considering the source API; you have to know *with which* things you are asking about compatibility.

在SemVer的定義中，有很多東西需要解讀，虛線三重版本號到底意味著什麼。這是一個承諾嗎？還是為一個版本選擇的版本號是一種估計值？也就是說，當 libbase 的維護者發布一個新版本，並選擇這是一個大版本、小版本還是補丁版本時，他們在說什麼？是否可以證明從 1.1.4 升級到 1.2.0 是安全且容易的，因為只有 API 的增加和錯誤的修正？當然不是。在 "簡單的 "API增加的情況下，libbase的不守規矩的使用者可能會做很多事情，導致建立中斷或行為改變。從根本上說，當只考慮源API時，你不能*證明*任何關於相容性的事情；你必須知道你在問*哪些*相容性的問題。

However, this idea of 『estimating』 compatibility begins to weaken when we talk about networks of dependencies and SAT-solvers applied to those networks. The fundamental problem in this formulation is the difference between node values in traditional SAT and version values in a SemVer dependency graph. A node in a three-SAT graph *is* either True or False. A version value (1.1.14) in a dependency graph is provided by the maintainer as an *estimate* of how compatible the new version is, given code that used the previous version. We」re building all of our version-satisfaction logic on top of a shaky foundation, treating estimates and self-attestation as absolute. As we」ll see, even if that works OK in limited cases, in the aggregate, it doesn」t necessarily have enough fidelity to underpin a healthy ecosystem.

然而，當我們談論依賴網路和應用於這些網路的SAT求解器時，這種 "預估"相容性的想法就開始弱化了。這種表述的基本問題是傳統SAT中的節點值和SemVer依賴關係圖中的版本值之間的區別。三種SAT圖中的節點*是*真或假。依賴關係圖中的版本值（1.1.14）是由維護者提供的，是對新版本的相容程度的*預估*，給定使用以前版本的程式碼。我們將所有的版本滿足邏輯建立在一個不穩定的基礎之上，將預估和自我證明視為絕對。正如我們將看到的，即使這在有限的情況下是可行的，但從總體上看，它不一定有足夠的模擬度來支撐一個健康的生態系統。

If we acknowledge that SemVer is a lossy estimate and represents only a subset of the possible scope of changes, we can begin to see it as a blunt instrument. In theory, it works fine as a shorthand. In practice, especially when we build SAT-solvers on top of it, SemVer can (and does) fail us by both overconstraining and underprotecting us.

如果我們承認SemVer是一個有損失的預估，並且只代表可能的變化範圍的一個子集，我們就可以開始把它看作是一個鈍器。在理論上，它作為一種速記工具是很好的。在實踐中，尤其是當我們在它上面建立SAT求解器時，SemVer可能（也確實）會因為過度約束和保護不足而讓我們失敗。

> [^9]: For example: a poorly implemented polyfill that adds the new libbase API ahead of time, causing a conflicting definition. Or, use of language reflection APIs to depend upon the precise number of APIs provided by libbase, introducing crashes if that number changes. These shouldn」t happen and are certainly rare even if they do happen by accident—the point is that the libbase providers can」t prove compatibility.
>
> 9  例如：一個實現不佳的 polyfill，提前新增了新的 libbase API，導致定義衝突。或者，使用語言反射 API 來依賴 libbase 提供的精確數量的 API，如果這個數量發生變化，就會引入崩潰。這些都不應該發生，而且即使是意外發生，也肯定很罕見--關鍵是 libbase 提供者無法證明相容性。

### SemVer Might Overconstrain  SemVer可能會過度限制

Consider what happens when libbase is recognized to be more than a single monolith: there are almost always independent interfaces within a library. Even if there are only two functions, we can see situations in which SemVer overconstrains us. Imagine that libbase is indeed composed of only two functions, Foo and Bar. Our mid-level dependencies liba and libb use only Foo. If the maintainer of libbase makes a breaking change to Bar, it is incumbent on them to bump the major version of lib base in a SemVer world. liba and libb are known to depend on libbase 1.x— SemVer dependency solvers won」t accept a 2.x version of that dependency. However, in reality these libraries would work together perfectly: only Bar changed, and that was unused. The compression inherent in 『I made a breaking change; I must bump the major version number』 is lossy when it doesn」t apply at the granularity of an individual atomic API unit. Although some dependencies might be fine grained enough for that to be accurate,[^10] that is not the norm for a SemVer ecosystem.

考慮一下當libbase被認定為不只是一個單體時會發生什麼：一個函式庫內幾乎都有獨立的介面。即使只有兩個函式，我們也可以看到 SemVer 對我們過度約束的情況。想象一下，libbase確實只由Foo和Bar這兩個函式組成。我們的中層依賴關係 liba 和 libb 只使用 Foo。如果 libbase 的維護者對 Bar 進行了破壞性的修改，那麼在 SemVer 世界中，他們就有責任提升 libbase 的主要版本。已知 liba 和 libb 依賴於 libbase 1.x——SemVer 依賴解決器不會接受這種依賴的 2.x 版本。然而，在現實中，這些函式庫可以完美地協同工作：只有Bar改變了，而且是未使用的。當 "我做了一個突破性的改變；我必須提高主要版本號"的固有壓縮不適用單個原子API單元的粒度時，它是有損的。雖然有些依賴關係可能足夠精細，所以這是很準確的，這不是SemVer生態系統的標準。

If SemVer overconstrains, either because of an unnecessarily severe version bump or insufficiently fine-grained application of SemVer numbers, automated package managers and SAT-solvers will report that your dependencies cannot be updated or installed, even if everything would work together flawlessly by ignoring the SemVer checks. Anyone who has ever been exposed to dependency hell during an upgrade might find this particularly infuriating: some large fraction of that effort was a complete waste of time.

如果SemVer過度約束，無論是由於不必要的嚴重的版本升級，還是由於對SemVer數字的應用不夠精細，自動軟體套件管理器和SAT求解器將報告你的依賴關係不能被更新或安裝，即使忽略SemVer檢查，一切都能完美地協同工作。任何曾經在升級過程中被暴露在依賴地獄中的人都會發現這一點特別令人生氣：其中很大一部分工作完全是浪費時間。

> [^10]: The Node ecosystem has noteworthy examples of dependencies that provide exactly one API.
>
> 10  節點生態系統有值得注意的依賴關係範例，這些依賴關係只提供一個API。

### SemVer Might Overpromise  SemVer可能過度承諾

On the flip side, the application of SemVer makes the explicit assumption that an API provider」s estimate of compatibility can be fully predictive and that changes fall into three buckets: breaking (by modification or removal), strictly additive, or non-API- impacting. If SemVer is a perfectly faithful representation of the risk of a change by classifying syntactic and semantic changes, how do we characterize a change that adds a one-millisecond delay to a time-sensitive API? Or, more plausibly: how do we characterize a change that alters the format of our logging output? Or that alters the order that we import external dependencies? Or that alters the order that results are returned in an 『unordered』 stream? Is it reasonable to assume that those changes are 『safe』 merely because those aren」t part of the syntax or contract of the API in question? What if the documentation said 『This may change in the future』? Or the API was named 『ForInternalUseByLibBaseOnlyDoNotTouchThisIReallyMeanIt?』[^11]

另一方面，SemVer的應用做出了明確的假設，即API提供者對相容性的預估可以完全預測，並且更改分為三個類：破壞（透過修改或刪除）、嚴格的新增或不影響API。如果SemVer透過對語法和語義變化進行分類，完全忠實地表示了變化的風險，那麼我們如何描述為時間敏感API增加一毫秒延遲的更改？或者，更合理的說法是：我們如何描述改變日誌輸出格式的更改？或者改變了我們匯入外部依賴關係的順序？或者改變了在 "無序"流中回傳結果的順序？僅僅因為這些變更不屬於問題中API的語法或契約的一部分，就認為這些變更是『安全的』是合理的嗎？如果文件中說"這在未來可能會發生變化"呢？或者API被命名為 "ForInternalUseByLibBaseOnlyDoNotTouchThisIReallyMeanIt？"

The idea that SemVer patch versions, which in theory are only changing implementation details, are 『safe』 changes absolutely runs afoul of Google」s experience with Hyrum」s Law—『With a sufficient number of users, every observable behavior of your system will be depended upon by someone.』 Changing the order that dependencies are imported, or changing the output order for an 『unordered』 producer will, at scale, invariably break assumptions that some consumer was (perhaps incorrectly) relying upon. The very term 『breaking change』 is misleading: there are changes that are theoretically breaking but safe in practice (removing an unused API). There are also changes that are theoretically safe but break client code in practice (any of our earlier Hyrum」s Law examples). We can see this in any SemVer/dependency-management system for which the version-number requirement system allows for restrictions on the patch number: if you can say liba requires libbase >1.1.14 rather than liba requires libbase 1.1, that」s clearly an admission that there are observable differences in patch versions.

SemVer補丁版本在理論上只是改變了實現細節，是 "安全"的更改，這種想法絕對違背了谷歌對Hyrum定律的經驗--"只要有足夠數量的使用者，你的系統的每一個可觀察到的行為都會被某人所依賴。" 改變依賴關係的匯入順序，或者改變一個 "無序"使用者的輸出順序，在規模上將不可避免地打破一些使用者（也許是錯誤地）所依賴的假設。"破壞性變化"這個術語本身就具有誤導性：有些更改在理論上是突破性的，但在實踐中是安全的（刪除未使用的API）。也有一些變化在理論上是安全的，但在實踐中會破壞用戶端程式碼（我們之前的任何一個Hyrum定律的例子）。我們可以在任何SemVer/依賴管理系統中看到這一點，其中的版本號要求系統允許對補丁號進行限制：如果你可以說liba需要libbase >1.1.14，而不是liba需要libbase 1.1，這顯然是承認補丁版本中存在明顯的差異。

*A change in isolation isn」t breaking or nonbreaking—*that statement can be evaluated only in the context of how it is being used. There is no absolute truth in the notion of 『This is a breaking change』; a change can been seen to be breaking for only a (known or unknown) set of existing users and use cases. The reality of how we evaluate a change inherently relies upon information that isn」t present in the SemVer formulation of dependency management: how are downstream users consuming this dependency?

*孤立的變化不是破壞性的，也不是非破壞性的*——這種說法只能在它被使用的情況下進行評估。在 "這是一個破壞性的變化"的概念中沒有絕對的真理；一個變化只能被看作是對（已知或未知的）現有使用者和用例的破壞。我們如何評估一個變化的現實，本質上依賴於SemVer制定的依賴管理中所沒有的訊息：下游使用者是如何使用這個依賴的？

Because of this, a SemVer constraint solver might report that your dependencies work together when they don」t, either because a bump was applied incorrectly or because something in your dependency network had a Hyrum」s Law dependence on something that wasn」t considered part of the observable API surface. In these cases, you might have either build errors or runtime bugs, with no theoretical upper bound on their severity.

正因為如此，SemVer約束求解器可能會報告說，你的依賴關係可以一起工作，但它們卻不能一起工作，這可能是因為錯誤地應用了一個坑點，或者是因為你的依賴網路中的某些東西與不被認為是可觀察API表面的一部分的東西存在Hyrum定律依賴。在這些情況下，你可能會有建立錯誤或執行時錯誤，其嚴重性在理論上沒有上限。

> [^11]: It」s worth noting: in our experience, naming like this doesn」t fully solve the problem of users reaching in to access private APIs. Prefer languages that have good control over public/private access to APIs of all forms.
>
> 11  值得注意的是：根據我們的經驗，這樣命名並不能完全解決使用者訪問私有API的問題。首選對所有形式的API的公共/私人訪問有良好控制的語言。

### Motivations  動機

There is a further argument that SemVer doesn」t always incentivize the creation of stable code. For a maintainer of an arbitrary dependency, there is variable systemic incentive to *not* make breaking changes and bump major versions. Some projects care deeply about compatibility and will go to great lengths to avoid a major-version bump. Others are more aggressive, even intentionally bumping major versions on a fixed schedule. The trouble is that most users of any given dependency are indirect users—they wouldn」t have any significant reasons to be aware of an upcoming change. Even most direct users don」t subscribe to mailing lists or other release notifications.

還有一種觀點認為，SemVer並不總是鼓勵建立穩定的程式碼。對於任意依賴的維護者來說，有一個可變的系統激勵機制來*不*做破壞性的修改和提升主要版本。一些專案非常關心相容性，並將竭盡全力避免出現重大版本衝突。其他專案則更加積極，甚至有意在一個固定的時間表上提升主要版本。問題是，任何給定依賴項的大多數使用者都是間接使用者——他們沒有任何重要的理由知道即將發生的更改。即使是最直接的使用者也不會訂閱郵件清單或其他發布通知。

All of which combines to suggest that no matter how many users will be inconvenienced by adoption of an incompatible change to a popular API, the maintainers bear a tiny fraction of the cost of the resulting version bump. For maintainers who are also users, there can also be an incentive *toward* breaking: it」s always easier to design a better interface in the absence of legacy constraints. This is part of why we think projects should publish clear statements of intent with respect to compatibility, usage, and breaking changes. Even if those are best-effort, nonbinding, or ignored by many users, it still gives us a starting point to reason about whether a breaking change/ major version bump is 『worth it,』 without bringing in these conflicting incentive structures.

所有這些都表明，不管有多少使用者會因為採用不相容的API而感到不便，維護者只需承擔由此帶來的版本升級的一小部分成本。對於同時也是使用者的維護者來說，也會有一個激勵機制，那就是：在沒有遺留限制的情況下，設計一個更好的介面總是更容易。這也是為什麼我們認為專案應該發表關於相容性、使用和破壞性變化的明確宣告的部分原因。即使這些都是盡力而為、不具約束力或被許多使用者忽略的，但它仍然為我們提供了一個起點，讓我們可以在不引入這些相互衝突的激勵結構的情況下，思考突破性的更改/重大版本升級是否『值得』。

[Go](https://research.swtch.com/vgo-import)and [Clojure](https://oreil.ly/Iq9f_)both handle this nicely: in their standard package management ecosystems, the equivalent of a major-version bump is expected to be a fully new package. This has a certain sense of justice to it: if you」re willing to break backward compatibility for your package, why do we pretend this is the same set of APIs? Repackaging and renaming everything seems like a reasonable amount of work to expect from a provider in exchange for them taking the nuclear option and throwing away backward compatibility.

[Go](https://research.swtch.com/vgo-import)和[Clojure](https://oreil.ly/Iq9f_)都很好地處理了這個問題：在他們的標準套件管理生態系統中，相當於一個主要版本的升級被認為是一個完全新的套件。這有一定的正義感：如果你願意為你的套件打破向後的相容性，為什麼我們要假裝這是同一套API？重新打包和重新命名一切似乎是一個合理的工作量，期望從提供者那裡得到，以換取他們接受核選項並拋棄向後相容性。

Finally, there」s the human fallibility of the process. In general, SemVer version bumps should be applied to *semantic* changes just as much as syntactic ones; changing the behavior of an API matters just as much as changing its structure. Although it」s plausible that tooling could be developed to evaluate whether any particular release involves syntactic changes to a set of public APIs, discerning whether there are meaningful and intentional semantic changes is computationally infeasible.[^12] Practically speaking, even the potential tools for identifying syntactic changes are limited. In almost all cases, it is up to the human judgement of the API provider whether to bump major, minor, or patch versions for any given change. If you」re relying on only a handful of professionally maintained dependencies, your expected exposure to this form of SemVer clerical error is probably low.[^13] If you have a network of thousands of dependencies underneath your product, you should be prepared for some amount of chaos simply from human error.

最後，還有過程中的人為失誤。一般來說，SemVer版本升級應該和語法變化一樣適用於*語義*變化；改變API的行為和改變其結構一樣重要。雖然開發工具來評估任何特定的版本是否涉及一組公共API的語法變化是可行的，但是要辨別是否存在有意義的、有意的語義變化在計算上是不可行的。實際上，即使是識別語法變化的潛在工具也是有限的。在幾乎所有的情況下，對於任何給定的變化，是否要碰撞主要版本、次要版本或補丁版本，都取決於API提供者的人為判斷。如果你只依賴少數幾個專業維護的依賴關係，那麼你對這種形式的SemVer文書錯誤的預期暴露可能很低。如果你的產品下面有成千上萬的依賴關係網路，你應該準備好接受某種程度的混亂，僅僅是因為人為錯誤。

> [^12]: In a world of ubiquitous unit tests, we could identify changes that required a change in test behavior, but it would still be difficult to algorithmically separate 『This is a behavioral change』 from 『This is a bug fix to a behavior that wasn」t intended/promised.』
>
> 12  在一個無處不在的單元測試的世界裡，我們可以識別需要改變測試行為的變化，但仍然很難在演算法上將 "這是一個行為上的變化 "與 "這是一個對不打算/承諾的行為的錯誤修復 "分開。
>
> [^13]: So, when it matters in the long term, choose well-maintained dependencies.
>
> 13  所以，當長期重要時，選擇維護良好的依賴關係。

### Minimum Version Selection  最小版本選擇

In 2018, as part of an essay series on building a package management system for the Go programming language, Google」s own Russ Cox described an interesting variation on SemVer dependency management: [Minimum Version Selection](https://research.swtch.com/vgo-mvs) (MVS). When updating the version for some node in the dependency network, it is possible that its dependencies need to be updated to newer versions to satisfy an updated SemVer requirement—this can then trigger further changes transitively. In most constraint- satisfaction/version-selection formulations, the newest possible versions of those downstream dependencies are chosen: after all, you」ll need to update to those new versions eventually, right?

2018年，作為為Go程式語言建立軟體套件管理系統的系列文章的一部分，谷歌自己的Russ Cox描述了SemVer依賴性管理的一個有趣變化。[最小版本選擇](https://research.swtch.com/vgo-mvs)（MVS）。當更新依賴網路中某個節點的版本時，它的依賴關係有可能需要更新到較新的版本，以滿足更新的SemVer需求——這可能會觸發進一步的變化。在大多數約束滿足/版本選擇公式中，這些下游依賴關係的最新版本被選中：畢竟，你最終需要更新到這些新版本，對嗎？

MVS makes the opposite choice: when liba」s specification requires libbase ≥1.7, we」ll try libbase 1.7 directly, even if a 1.8 is available. This 『produces high-fidelity builds in which the dependencies a user builds are as close as possible to the ones the author developed against.』[^14] There is a critically important truth revealed in this point: when liba says it requires libbase ≥1.7, that almost certainly means that the developer of liba had libbase 1.7 installed. Assuming that the maintainer performed even basic testing before publishing,[^15] we have at least anecdotal evidence of interoperability testing for that version of liba and version 1.7 of libbase. It」s not CI or proof that everything has been unit tested together, but it」s something.

MVS做出了相反的選擇：當liba的規範要求libbase≥1.7時，我們會直接嘗試libbase 1.7，即使有1.8的版本。這 "產生了高模擬的建立，其中使用者建立的依賴關係儘可能地接近作者開發的依賴關係。"在這一點上揭示了一個極其重要的事實：當liba說它需要libbase≥1.7時，這幾乎肯定意味著liba的開發者安裝了libbase 1.7。假設維護者在發布之前進行了哪怕是基本的測試，我們至少有關於該版本的liba和libbase版本1.7的互操作性測試的軼事證據。這不是CI，也不能證明所有的東西都一起進行了單元測試，但它是有意義的。

Absent accurate input constraints derived from 100% accurate prediction of the future, it」s best to make the smallest jump forward possible. Just as it」s usually safer to commit an hour of work to your project instead of dumping a year of work all at once, smaller steps forward in your dependency updates are safer. MVS just walks forward each affected dependency only as far as is required and says, 『OK, I」ve walked forward far enough to get what you asked for (and not farther). Why don」t you run some tests and see if things are good?』

在沒有100%準確預測未來而產生的準確輸入約束的情況下，最好是儘可能地向前跳躍。正如將一小時的工作投入到專案中通常比一次完成一年的工作更安全一樣，依賴項更新中的小步驟也更安全。MVS只是在每個受影響的依賴關係上向前走了一段距離，然後說："好的，我已經向前走了一段距離，足以得到你所要求的東西（而不是更遠）。你為什麼不執行一些測試，看看情況是否良好？"

Inherent in the idea of MVS is the admission that a newer version might introduce an incompatibility in practice, even if the version numbers *in theory* say otherwise. This is recognizing the core concern with SemVer, using MVS or not: there is some loss of fidelity in this compression of software changes into version numbers. MVS gives some additional practical fidelity, trying to produce selected versions closest to those that have presumably been tested together. This might be enough of a boost to make a larger set of dependency networks function properly. Unfortunately, we haven」t found a good way to empirically verify that idea. The jury is still out on whether MVS makes SemVer 『good enough』 without fixing the basic theoretical and incentive problems with the approach, but we still believe it represents a manifest improvement in the application of SemVer constraints as they are used today.

在MVS的理念中，承認較新的版本在實踐中可能會帶來不相容，即使版本號在*理論上*說不相容。這就是認識到SemVer的核心問題，無論是否使用MVS：在將軟體更改壓縮為版本號的過程中，模擬度有所損失。MVS提供了一些額外的實際模擬度，試圖產生最接近那些可能已經被一起測試過的版本的選定版本。這可能是一個足夠的推動力，使更大的依賴網路正常運作。不幸的是，我們還沒有找到一個很好的方法來經驗性地驗證這個想法。MVS是否能在不解決該方法的基本理論和激勵問題的情況下使SemVer『足夠好』還沒有定論，但我們仍然認為，它代表了SemVer約束應用的一個明顯改進，正如今天所使用的那樣。

> [^14]: Russ Cox, 『Minimal Version Selection,』 February 21, 2018, https://research.swtch.com/vgo-mvs.
>
> 14 Russ Cox，"最小的版本選擇"，2018年2月21日，https://research.swtch.com/vgo-mvs。
> 
> [^15]: If that assumption doesn」t hold, you should really stop depending on liba.
>
> 15 如果這個假設不成立，你真的應該停止對liba的依賴。

### So, Does SemVer Work? 那麼，SemVer是否有效？

SemVer works well enough in limited scales. It」s deeply important, however, to recognize what it is actually saying and what it cannot. SemVer will work fine provided that:

- Your dependency providers are accurate and responsible (to avoid human error in SemVer bumps)
- Your dependencies are fine grained (to avoid falsely overconstraining when unused/unrelated APIs in your dependencies are updated, and the associated risk of unsatisfiable SemVer requirements)
- All usage of all APIs is within the expected usage (to avoid being broken in surprising fashion by an assumed-compatible change, either directly or in code you depend upon transitively)

SemVer在有限的範圍內執行良好。然而，認識到它實際上在做什麼，以及它不能做什麼，是非常重要的。SemVer將工作得很好，前提是:

- 你的依賴關係提供者準確且負責（以避免SemVer衝突中的人為錯誤）
- 你的依賴關係是細粒度的（以避免在更新依賴關係中未使用/不相關的API時錯誤地過度約束，以及不可滿足SemVer需求的相關風險）。
- 所有API的所有使用都在預期的使用範圍內（以避免被一個假定的相容變化以令人驚訝的方式破壞，無論是直接的還是在你轉場依賴的程式碼中）。

When you have only a few carefully chosen and well-maintained dependencies in your dependency graph, SemVer can be a perfectly suitable solution.

當你的依賴關係中只有少數精心選擇和維護良好的依賴關係時，SemVer可以成為一個完全合適的解決方案。

However, our experience at Google suggests that it is unlikely that you can have *any* of those three properties at scale and keep them working constantly over time. Scale tends to be the thing that shows the weaknesses in SemVer. As your dependency network scales up, both in the size of each dependency and the number of dependencies (as well as any monorepo effects from having multiple projects depending on the same network of external dependencies), the compounded fidelity loss in SemVer will begin to dominate. These failures manifest as both false positives (practically incompatible versions that theoretically should have worked) and false negatives (compatible versions disallowed by SAT-solvers and resulting dependency hell).

然而，我們在谷歌的經驗表明，你不太可能在規模上擁有這三個屬性中的任何一個，並且隨著時間的推移保持它們持續工作。規模往往是顯示SemVer弱點的東西。隨著你的依賴網路規模的擴大，無論是每個依賴的規模還是依賴的數量（以及由多個專案依賴於同一外部依賴網路而產生的任何單一效應），SemVer的複合模擬度損失將開始佔據主導地位。這些故障表現為誤報（理論上應該有效的實際不相容版本）和漏報（SAT求解器不允許的相容版本以及由此產生的依賴地獄）。

## Dependency Management with Infinite Resources  無限資源下的依賴管理

Here」s a useful thought experiment when considering dependency-management solutions: what would dependency management look like if we all had access to infinite compute resources? That is, what」s the best we could hope for, if we aren」t resource constrained but are limited only by visibility and weak coordination among organizations? As we see it currently, the industry relies on SemVer for three reasons:

- It requires only local information (an API provider doesn」t *need* to know the particulars of downstream users)
- It doesn」t assume the availability of tests (not ubiquitous in the industry yet, but definitely moving that way in the next decade), compute resources to run the tests, or CI systems to monitor the test results
- It」s the existing practice

在考慮依賴管理解決方案時，有一個有用的思想實驗：如果我們都能獲得無限的計算資源，依賴管理會是什麼樣子？也就是說，如果我們不受資源限制，而只受限於組織間的可見性和弱協調性，那麼我們能希望的最好結果是什麼？正如我們目前所看到的，該行業依賴SemVer的原因有三個：

- 它只需要本地訊息（API提供者不需要知道下游使用者的識別符號）
- 它不需要測試的可用性（在行業中還沒有普及，但在未來十年肯定會向這個方向發展）、執行測試的計算資源或監控測試結果的CI系統的可用性。
- 這是現成的做法

The  『requirement』  of  local  information  isn」t  really  necessary,  specifically  because dependency networks tend to form in only two environments:

- Within a single organization
- Within the OSS ecosystem, where source is visible even if the projects are not necessarily collaborating

對本地訊息的 "要求 "並不是真正必要的，特別是因為依賴性網路往往只在兩種環境中形成：

- 在一個組織內
- 在開放原始碼軟體生態系統內，即使專案不一定合作，原始碼也是可見的

In either of those cases, significant information about downstream usage is *available*, even if it isn」t being readily exposed or acted upon today. That is, part of SemVer」s effective dominance is that we」re choosing to ignore information that is theoretically available to us. If we had access to more compute resources and that dependency information was surfaced readily, the community would probably find a use for it.

在這兩種情況下，關於下游使用情況的重要訊息是*可用的*，目前還沒有暴露或採取行動。也就是說，SemVer的有效主導地位的部分原因是我們選擇忽略了理論上我們可以獲得的訊息。如果我們能夠獲得更多的計算資源，並且依賴性訊息能夠很容易地浮出水面，社群可能會發現它的用途。

Although an OSS package can have innumerable closed-source dependents, the common case is that popular OSS packages are popular both publicly and privately. Dependency networks don」t (can」t) aggressively mix public and private dependencies: generally, there is a public subset and a separate private subgraph.[^16]

雖然一個開放原始碼軟體套件可以有無數的閉源依賴，但常見的情況是，受歡迎的開放原始碼軟體套件在公開和私下裡都很受歡迎。依賴網路不會（不能）積極地混合公共和私人依賴關係：通常，有一個公共子集和一個單獨的私有子集。

Next, we must remember the *intent* of SemVer: 『In my estimation, this change will be easy (or not) to adopt.』 Is there a better way of conveying that information? Yes, in the form of practical experience demonstrating that the change is easy to adopt. How do we get such experience? If most (or at least a representative sample) of our dependencies are publicly visible, we run the tests for those dependencies with every proposed change. With a sufficiently large number of such tests, we have at least a statistical argument that the change is safe in the practical Hyrum」s-Law sense. The tests still pass, the change is good—it doesn」t matter whether this is API impacting, bug fixing, or anything in between; there」s no need to classify or estimate.

接下來，我們必須記住SemVer的*意圖*："據我估計，這種變化將很容易（或不容易）被採納。" 是否有更好的方式來傳達這一訊息？是的，以實踐經驗的形式，證明該變化是容易採用的。我們如何獲得這種經驗呢？如果我們大部分（或者至少是有代表性的樣本）的依賴關係是公開的，那麼我們就在每一個提議的改變中對這些依賴關係進行測試。有了足夠多的這樣的測試，我們至少有了一個統計學上的論據，即從實際的Hyrum定律意義上來說，這個變化是安全的。測試仍然透過，變化就是好的——這與影響API、修復bug或介於兩者之間的事情無關；沒有必要進行分類或評估。

Imagine, then, that the OSS ecosystem moved to a world in which changes were accompanied with *evidence* of whether they are safe. If we pull compute costs out of the equation, the *truth*[^17] of 『how safe is this』 comes from running affected tests in downstream dependencies.

想象一下，開放原始碼軟體的生態系統轉向一個變化伴隨著*證據*的世界，即它們是否安全。如果我們把計算成本排除在外，那麼 "這有多安全 "的*真相*來自於在下游依賴關係中執行受影響的測試。

Even without formal CI applied to the entire OSS ecosystem, we can of course use such a dependency graph and other secondary signals to do a more targeted presubmit analysis. Prioritize tests in dependencies that are heavily used. Prioritize tests in dependencies that are well maintained. Prioritize tests in dependencies that have a history of providing good signal and high-quality test results. Beyond just prioritizing tests based on the projects that are likely to give us the most information about experimental change quality, we might be able to use information from the change authors to help estimate risk and select an appropriate testing strategy. Running 『all affected』 tests is theoretically necessary if the goal is 『nothing that anyone relies upon is change in a breaking fashion.』 If we consider the goal to be more in line with 『risk mitigation,』 a statistical argument becomes a more appealing (and cost-effective) approach.

即使沒有正式的CI應用於整個開放原始碼生態系統，我們當然也可以使用這樣的依賴關係和其他次級訊號來做更有針對性的預送出分析。優先考慮大量使用的依賴項中的測試。優先考慮維護良好的依賴項中的測試。優先考慮那些有提供良好訊號和高質量測試結果歷史的依賴關係中的測試。除了根據有可能給我們提供最多實驗性變化質量訊息的專案來確定測試的優先級外，我們還可以利用變化作者的訊息來幫助估計風險和選擇適當的測試策略。如果目標是 任何人所依賴的都是一種破壞性的改變"，執行 "所有受影響 "的測試在理論上是必要的。如果我們認為目標更符合 "風險緩解"，那麼統計論證就會成為一種更有吸引力（和成本效益）的方法。

In Chapter 12, we identified four varieties of change, ranging from pure refactorings to modification of existing functionality. Given a CI-based model for dependency updating, we can begin to map those varieties of change onto a SemVer-like model for which the author of a change estimates the risk and applies an appropriate level of testing. For example, a pure refactoring change that modifies only internal APIs might be assumed to be low risk and justify running tests only in our own project and perhaps a sampling of important direct dependents. On the other hand, a change that removes a deprecated interface or changes observable behaviors might require as much testing as we can afford.

在第12章中，我們確定了四種變化，從純粹的重構到對現有功能的修改。考慮到基於CI的依賴更新模型，我們可以開始將這些變化種類對映到類似SemVer的模型上，對於這些變化，變更的作者會估計風險並應用適當的測試水平。例如，僅修改內部API的純重構變化可能被認為是低風險的，並證明僅在我們自己的專案和重要的直接依賴者中執行測試。另一方面，刪除一個廢棄的介面或改變可觀察到的行為的變化可能需要我們進行儘可能多的測試。

What changes would we need to the OSS ecosystem to apply such a model? Unfortunately, quite a few:

- All dependencies must provide unit tests. Although we are moving inexorably toward a world in which unit testing is both well accepted and ubiquitous, we are not there yet.
- The dependency network for the majority of the OSS ecosystem is understood. It is unclear that any mechanism is currently available to perform graph algorithms on that network—the information is *public* and *available,* but not actually generally indexed or usable. Many package-management systems/dependency- management ecosystems allow you to see the dependencies of a project, but not the reverse edges, the dependents.
- The availability of compute resources for executing CI is still very limited. Most developers don」t have access to build-and-test compute clusters.
- Dependencies are often expressed in a pinned fashion. As a maintainer of libbase, we can」t experimentally run a change through the tests for liba and libb if those dependencies are explicitly depending on a specific pinned version of libbase.
- We might want to explicitly include history and reputation in CI calculations. A proposed change that breaks a project that has a longstanding history of tests continuing to pass gives us a different form of evidence than a breakage in a project that was only added recently and has a history of breaking for unrelated reasons.

為了應用這樣的模式，我們需要對開放原始碼軟體的生態系統進行哪些改變？不幸的是，相當多:

- 所有的依賴項必須提供單元測試。儘管我們正不可阻擋地走向一個單元測試被廣泛接受和無處不在的世界，但我們還沒有到那一步。
- 瞭解大多數開放原始碼軟體生態系統的依賴網路。目前尚不清楚是否有任何機制可用於在該網路上執行圖形演算法--訊息是公開的，可用的，但實際上沒有被普遍索引或使用。許多軟體套件管理系統/依賴性管理生態系統允許你看到一個專案的依賴項，但不允許檢視反向邊緣和依賴關係。
- 用於執行CI的計算資源的可用性仍然非常有限。大多數開發者沒有機會使用建立和測試的計算叢集。
- 依賴項通常以固定方式表示。作為libbase的維護者，如果liba和libb的依賴項顯式地依賴於libbase的特定固定版本，那麼我們就不能透過liba和libb的測試實驗性地執行更改。
- 我們可能希望在CI計算中明確包括歷史和聲譽。一個提議的變更打破了一個長期以來一直透過測試的專案，這給我們提供了一種不同形式的證據，而不是一個最近才新增的專案中的破壞，並且由於不相關的原因而有破壞的歷史。

Inherent in this is a scale question: against which versions of each dependency in the network do you test presubmit changes? If we test against the full combination of all historical versions, we」re going to burn a truly staggering amount of compute resources, even by Google standards. The most obvious simplification to this version- selection strategy would seem to be 『test the current stable version』 (trunk-based development is the goal, after all). And thus, the model of dependency management given infinite resources is effectively that of the Live at Head model. The outstanding question is whether that model can apply effectively with a more practical resource availability and whether API providers are willing to take greater responsibility for testing the practical safety of their changes. Recognizing where our existing low-cost facilities are an oversimplification of the difficult-to-compute truth that we are looking for is still a useful exercise.

這裡面有一個規模問題：你要針對網路中每個依賴關係的哪些版本來測試預送出的變化？如果我們針對所有歷史版本的完整組合進行測試，我們將消耗大量的計算資源，即使按照谷歌的能力。這個版本選擇策略最明顯的簡化似乎是 "測試當前的穩定版本"（畢竟，基於主幹的開發是目標）。因此，在資源無限的情況下，依賴管理的模式實際上就是 "Live at Head"的模式。懸而未決的問題是，該模型是否可以有效地適用於更實際的資源可用性，以及API提供者是否願意承擔更大的責任來測試其變化的實際安全性。認識到我們現有的低成本設施是對我們正在尋找的難以計算的真相的過度簡化，仍然是一項有益的工作。

> [^16]: Because the public OSS dependency network can」t generally depend on a bunch of private nodes, graphics firmware notwithstanding.
>
> 16 因為公共開放原始碼軟體的依賴網路一般不能依賴一堆私人節點，儘管有圖形固定。

> [^17]: Or something very close to it.
>
> 17 或者是非常接近於此的東西。

### Exporting Dependencies  匯出依賴

So far, we」ve only talked about taking on dependencies; that is, depending on software that other people have written. It」s also worth thinking about how we build software that can be *used* as a dependency. This goes beyond just the mechanics of packaging software and uploading it to a repository: we need to think about the benefits, costs, and risks of providing software, for both us and our potential dependents.

到目前為止，我們只討論了依賴；也就是說，這取決於其他人編寫的軟體。同樣值得思考的是，我們如何建立可以作為依賴使用的軟體。這不僅僅是打包軟體並將其上傳到儲存庫的機制：我們需要考慮提供軟體的好處、成本和風險，對我們和我們的潛在依賴者都是如此。

There are two major ways that an innocuous and hopefully charitable act like 『open sourcing a library』 can become a possible loss for an organization. First, it can eventually become a drag on the reputation of your organization if implemented poorly or not maintained properly. As the Apache community saying goes, we ought to prioritize 『community over code.』 If you provide great code but are a poor community member, that can still be harmful to your organization and the broader community. Second, a well-intentioned release can become a tax on engineering efficiency if you can」t keep things in sync. Given time, all forks will become expensive.

像 "開源庫"這樣無害的慈善的行為，有兩種主要方式可以成為一個組織的可能損失。首先，如果實施不力或維護不當，它最終會拖累你的組織的聲譽。正如Apache社群的說法，我們應該優先考慮 "社群優先於程式碼"。如果你提供了很好的程式碼，但卻是一個糟糕的社群成員，這仍然會對你的組織和更廣泛的社群造成傷害。其次，如果你不能保持同步，一個善意的發布會成為對工程效率的一種負擔。只要有時間，所有的分支都會變得沉重。

#### Example: open sourcing gflags  範例：開源GFLAG

For reputation loss, consider the case of something like Google」s experience circa 2006 open sourcing our C++ command-line flag libraries. Surely giving back to the open source community is a purely good act that won」t come back to haunt us, right? Sadly, no. A host of reasons conspired to make this good act into something that certainly hurt our reputation and possibly damaged the OSS community as well:

- At the time, we didn」t have the ability to execute large-scale refactorings, so everything that used that library internally had to remain exactly the same—we couldn」t move the code to a new location in the codebase.
- We segregated our repository into 『code developed in-house』 (which can be copied freely if it needs to be forked, so long as it is renamed properly) and 『code that may have legal/licensing concerns』 (which can have more nuanced usage requirements).
- If an OSS project accepts code from outside developers, that」s generally a legal issue—the project originator doesn」t *own* that contribution, they only have rights to it.

對於信譽的損失，可以考慮像谷歌在2006年左右開放我們的C++指令行標誌庫的經驗的情況。當然，回饋開源社群是一個純粹的善舉，不會回傳困擾我們，對嗎？遺憾的是，不是。有很多原因共同促使這一善舉變成了肯定會傷害我們的聲譽，也可能會損害開放原始碼社群:

- 當時，我們沒有能力進行大規模的重構，所以所有內部使用該函式庫的東西都必須保持相同——我們不能把程式碼移到程式碼庫的新位置。
- 我們將我們的資源庫隔離成 "內部開發的程式碼"（如果需要分支，可以自由複製，只要正確重新命名）和 "可能有法律/許可問題的程式碼"（可能有更細微的使用要求）。
- 如果一個開放原始碼軟體專案接受來自外部開發者的程式碼，這通常是一個法律問題——專案發起人並不*擁有*該貢獻，他們只擁有對它的使用權利。

As a result, the gflags project was doomed to be either a 『throw over the wall』 release or a disconnected fork. Patches contributed to the project couldn」t be reincorporated into the original source inside of Google, and we couldn」t move the project within our monorepo because we hadn」t yet mastered that form of refactoring, nor could we make everything internally depend on the OSS version.

因此，gflags 專案註定是一個 "丟擲"的版本，或者是一個斷開的分支。貢獻給專案的補丁不能被重新納入谷歌內部的原始原始碼，我們也無法將該專案轉移到monorepo中，因為我們還沒有掌握這種重構形式，也無法讓內部的一切都依賴於開放原始碼版本。

Further, like most organizations, our priorities have shifted and changed over time. Around the time of the original release of that flags library, we were interested in  products outside of our traditional space (web applications, search), including things like Google Earth, which had a much more traditional distribution mechanism: precompiled binaries for a variety of platforms. In the late 2000s, it was unusual but not unheard of for a library in our monorepo, especially something low-level like flags, to be used on a variety of platforms. As time went on and Google grew, our focus narrowed to the point that it was extremely rare for any libraries to be built with anything other than our in-house configured toolchain, then deployed to our production fleet. The 『portability』 concerns for properly supporting an OSS project like flags were nearly impossible to maintain: our internal tools simply didn」t have support for those platforms, and our average developer didn」t have to interact with external tools. It was a constant battle to try to maintain portability.

此外，像大多數組織一樣，我們的優先事項隨著時間的推移而發生了改變。在最初發布flags函式庫的時候，我們對傳統領域（網路應用、搜尋）以外的產品感興趣，包括像谷歌地球這樣的產品，它有一個更傳統的發布機制：為各種平台預編譯的二進位檔案。在21世紀末，在我們的monorepo中的一個函式庫，特別是像flags這樣的低階的東西，被用在各種平台上，這是不正常的，但也不是沒有。隨著時間的推移和谷歌的成長，我們的關注點逐漸縮小，除了我們內部設定的工具鏈之外，很少有任何函式庫是用其他東西建立的，然後部署到我們的生產機群。對於正確支援像flags這樣的開放原始碼軟體專案來說，"可移植性"問題幾乎是不可能維持的：我們的內部工具根本沒有對這些平台的支援，而我們的普通開發人員也不需要與外部工具進行互動。為了保持可移植性，這是一場持久戰。

As the original authors and OSS supporters moved on to new companies or new teams, it eventually became clear that nobody internally was really supporting our OSS flags project—nobody could tie that support back to the priorities for any particular team. Given that it was no specific team」s job, and nobody could say why it was important, it isn」t surprising that we basically let that project rot externally.[^18] The internal and external versions diverged slowly over time, and eventually some external developers took the external version and forked it, giving it some proper attention.

隨著最初的作者和開放原始碼軟體支援者轉到新的公司或新的團隊，最終很明顯，內部沒有人真正支援我們的開放原始碼軟體flags專案——沒有人能夠將這種支援與任何特定團隊的優先事項聯絡起來。考慮到這不是特定團隊的工作，也沒人能說清楚為什麼它很重要，我們基本上讓這個專案在外部爛掉也就不奇怪了。隨著時間的推移，內部和外部的版本慢慢發生了分歧，最終一些外部開發者把外部的版本拆分，給了它一些適當的關注。

Other than the initial 『Oh look, Google contributed something to the open source world,』 no part of that made us look good, and yet every little piece of it made sense given the priorities of our engineering organization. Those of us who have been close to it have learned, 『Don」t release things without a plan (and a mandate) to support it for the long term.』 Whether the whole of Google engineering has learned that or not remains to be seen. It」s a big organization.

除了最初的『哦，看，谷歌為開源世界做出了一些貢獻』之外，沒有任何一部分能讓我們看起來很好，但考慮到我們工程組織的優先事項，它的每一個小部分都是有意義的。我們這些與它關係密切的人已經瞭解到，『在沒有長期支援它的計劃（和授權）的情況下，不要發布任何東西。』整個谷歌工程部門是否已經瞭解到這一點還有待觀察。這是一個大組織。

Above and beyond the nebulous 『We look bad,』 there are also parts of this story that illustrate how we can be subject to technical problems stemming from poorly released/poorly maintained external dependencies. Although the flags library was shared but ignored, there were still some Google-backed open source projects, or projects that needed to be shareable outside of our monorepo ecosystem. Unsurprisingly, the authors of those other projects were able to identify[^19] the common API subset between the internal and external forks of that library. Because that common subset stayed fairly stable between the two versions for a long period, it silently became 『the way to do this』 for the rare teams that had unusual portability requirements between roughly 2008 and 2017. Their code could build in both internal and external ecosystems, switching out forked versions of the flags library depending on environment.

除了模糊的『我們看起來很糟糕』之外，這個故事中還有一些部分說明了我們如何受到由於發布/維護不當的外部依賴關係而產生的技術問題的影響。雖然flags函式庫是共享的，但被忽略了，但仍然有一些由Google支援的開源專案，或者需要在monorepo生態系統之外共享的專案。毫不奇怪，這些其他專案的作者能夠識別該函式庫內部和外部分支之間的公共API子集。由於該通用子集在兩個版本之間保持了相當長的一段時間的穩定，因此它悄悄地成為了在2008年到2017年間具有不同尋常的可移植性需求的少數團隊的『實現方法』。他們的程式碼可以在內部和外部生態系統中建立，根據環境的不同，可以切換出flags函式庫的分支版本。

Then, for unrelated reasons, C++ library teams began tweaking observable-but-not- documented pieces of the internal flag implementation. At that point, everyone who was depending on the stability and equivalence of an unsupported external fork started screaming that their builds and releases were suddenly broken. An optimization opportunity worth some thousands of aggregate CPUs across Google」s fleet was significantly delayed, not because it was difficult to update the API that 250 million lines of code depended upon, but because a tiny handful of projects were relying on unpromised and unexpected things. Once again, Hyrum」s Law affects software changes, in this case even for forked APIs maintained by separate organizations.

然後，由於不相關的原因，C++函式庫團隊開始調整內部標誌實現中可觀察到但沒有記錄的部分。在這一點上，所有依賴於不支援的外部分支的穩定性和等效性的人都開始尖叫，他們的建立和發布突然被破壞。一個值得在谷歌叢集中使用數千個CPU的最佳化機會被大大推遲了，不是因為難以更新2.5億行程式碼所依賴的API，而是因為極少數專案依賴於未經預測和意外的東西。Hyrum定律再一次影響了軟體的變化，在這種情況下，甚至是由不同組織維護的分叉API。

> [^18]: That isn」t to say it」s right or wise, just that as an organization we let some things slip through the cracks.
>
> 18  這並不是說這是對的或明智的，只是作為一個組織，我們讓一些事情從縫隙中溜走。
> 
> [^19]: Often through trial and error.
>
> 19 往往是透過試驗和錯誤。

----
#### Case Study: AppEngine  案例研究：AppEngine

A more serious example of exposing ourselves to greater risk of unexpected technical dependency comes from publishing Google」s AppEngine service. This service allows users to write their applications on top of an existing framework in one of several popular programming languages. So long as the application is written with a proper storage/state management model, the AppEngine service allows those applications to scale up to huge usage levels: backing storage and frontend management are managed and cloned on demand by Google」s production infrastructure.

一個更嚴重的技術依賴將我們自己暴露在意料外的更大風險中的例子來自於發布谷歌的AppEngine服務。這項服務允許使用者在現有框架的基礎上用幾種流行的程式語言之一編寫他們的應用程式。只要應用程式是用適當的儲存/狀態管理模型編寫的，AppEngine服務允許這些應用程式擴充套件到超大規模的使用水平：備份儲存和前端管理是由谷歌的生產基礎設施按需管理和複製的。

Originally, AppEngine」s support for Python was a 32-bit build running with an older version of the Python interpreter. The AppEngine system itself was (of course) implemented in our monorepo and built with the rest of our common tools, in Python and in C++ for backend support. In 2014 we started the process of doing a major update to the Python runtime alongside our C++ compiler and standard library installations, with the result being that we effectively tied 『code that builds with the current C++ compiler』 to 『code that uses the updated Python version』—a project that upgraded one of those dependencies inherently upgraded the other at the same time. For most projects, this was a non-issue. For a few projects, because of edge cases and Hyrum」s Law, our language platform experts wound up doing some investigation and debugging to unblock the transition. In a terrifying instance of Hyrum」s Law running into business practicalities, AppEngine discovered that many of its users, our paying customers, couldn」t (or wouldn」t) update: either they didn」t want to take the change to the newer Python version, or they couldn」t afford the resource consumption changes involved in moving from 32-bit to 64-bit Python. Because there were some customers that were paying a significant amount of money for AppEngine services, AppEngine was able to make a strong business case that a forced switch to the new language and compiler versions must be delayed. This inherently meant that every piece of C++ code in the transitive closure of dependencies from AppEngine had to be compatible with the older compiler and standard library versions: any bug fixes or performance optimizations that could be made to that infrastructure had to be compatible across versions. That situation persisted for almost three years.

最初，AppEngine對Python的支援是使用舊版本的Python直譯器執行的32位建立。AppEngine系統本身（當然）是在我們的monorepo中實現的，並與我們其他的通用工具一起建立，用Python和C++來支援後端。2014年，我們開始對Python執行時進行重大更新，同時安裝C++編譯器和標準庫，其結果是我們有效地將 "用當前C++編譯器建立的程式碼 "與 "使用更新的Python版本的程式碼 "聯絡起來——一個專案如果升級了這些依賴中的一個，就同時升級了另一個。對於大多數專案來說，這並不是一個問題。對於少數專案，由於邊緣案例和海勒姆定律，我們的語言平台專家最終做了一些調查和除錯，以解除轉場的障礙。在一個可怕的海勒姆定律與商業實際相結合的例子中，AppEngine發現它的許多使用者，即我們的付費客戶，不能（或不願）更新：要麼他們不想改變到較新的Python版本，要麼他們負擔不起從32位到64位Python的資源消耗變化。因為有一些客戶為AppEngine的服務支付了大量的費用，AppEngine能夠提出一個強有力的商業方案，即必須推遲強制切換到新的語言和編譯器版本。這就意味著AppEngine的依賴關係中的每一段C++程式碼都必須與舊的編譯器和標準庫版本相容：對該基礎設施的任何錯誤修復或效能最佳化都必須跨版本相容。這種情況持續了近三年。

-----

With enough users, any 『observable』 of your system will come to be depended upon by somebody. At Google, we constrain all of our internal users within the boundaries of our technical stack and ensure visibility into their usage with the monorepo and code indexing systems, so it is far easier to ensure that useful change remains possible. When we shift from source control to dependency management and lose visibility into how code is used or are subject to competing priorities from outside groups (especially ones that are paying you), it becomes much more difficult to make pure engineering trade-offs. Releasing APIs of any sort exposes you to the possibility of competing priorities and unforeseen constraints by outsiders. This isn」t to say that you shouldn」t release APIs; it serves only to provide the reminder: external users of an API cost a lot more to maintain than internal ones.

有了足夠多的使用者，你的系統的任何 "可觀察到的"都會被某些人所依賴。在谷歌，我們把所有的內部使用者都限制在我們的技術堆疊的範圍內，並透過monorepo和程式碼索引系統確保對他們的使用情況的可見性，所以更容易確保有用的改變是可能的。當我們從原始碼控制轉向依賴管理，並失去了對程式碼使用情況的可見性，或者受到來自外部團體（尤其是那些付錢給你的團體）的高優先級的影響時，要做出純粹的工程權衡就變得更加困難。發布任何型別的API都會使你暴露在競爭性的優先級和外部人員不可預見的限制的可能性中。這並不是說你不應該發布API；這只是為了提醒你：API的外部使用者比內部使用者的維護成本高得多。

Sharing code with the outside world, either as an open source release or as a closed- source library release, is not a simple matter of charity (in the OSS case) or business opportunity (in the closed-source case). Dependent users that you cannot monitor, in different organizations, with different priorities, will eventually exert some form of Hyrum」s Law inertia on that code. Especially if you are working with long timescales, it is impossible to accurately predict the set of necessary or useful changes that could become valuable. When evaluating whether to release something, be aware of the long-term risks: externally shared dependencies are often much more expensive to modify over time.

與外界分享程式碼，無論是作為開放原始碼發布還是作為閉源庫發布，都不是一個簡單的慈善問題（在開放原始碼的情況下）或商業機會（在閉源的情況下）。你無法監控的依賴使用者，在不同的組織中，有不同的優先級，最終會對該程式碼施加某種形式的海勒姆定律的慣性。特別是當你工作的時間尺度較長時，你不可能準確地預測可能成為有價值的必要或有用的變化的集合。當評估是否要發布一些東西時，要意識到長期的風險：外部共享的依賴關係隨著時間的推移，修改的成本往往要高得多。

## Conclusion  總結

Dependency management is inherently challenging—we」re looking for solutions to management of complex API surfaces and webs of dependencies, where the maintainers of those dependencies generally have little or no assumption of coordination. The de facto standard for managing a network of dependencies is semantic versioning, or SemVer, which provides a lossy summary of the perceived risk in adopting any particular change. SemVer presupposes that we can a priori predict the severity of a change, in the absence of knowledge of how the API in question is being consumed: Hyrum」s Law informs us otherwise. However, SemVer works well enough at small scale, and even better when we include the MVS approach. As the size of the dependency network grows, Hyrum」s Law issues and fidelity loss in SemVer make managing the selection of new versions increasingly difficult.

依賴管理在本質上是一種挑戰--我們正在尋找管理複雜的API表面和依賴關係網路的解決方案，這些依賴關係的維護者通常很少或根本沒有協調的假設。管理依賴關係網路的事實上的標準是語義版本管理（SemVer），它對採用任何特定變化的感知風險提供了有損的總結。SemVer的前提是，在不知道有關的API是如何被消費的情況下，我們可以先驗地預測變化的嚴重性。海勒姆定律告訴我們並非如此。然而，SemVer在小規模下工作得足夠好，當我們包括MVS方法時，甚至更好。隨著依賴網路規模的擴大，SemVer中的Hyrum定律問題和保真度損失使得管理新版本的選擇越來越困難。

It is possible, however, that we move toward a world in which maintainer-provided estimates of compatibility (SemVer version numbers) are dropped in favor of experience-driven evidence: running the tests of affected downstream packages. If API providers take greater responsibility for testing against their users and clearly advertise what types of changes are expected, we have the possibility of higher-fidelity dependency networks at even larger scale.

然而，我們有可能走向這樣一個世界：維護者提供的相容性估計（SemVer版本號）被放棄，而採用經驗驅動的證據：執行受影響的下游套件的測試。如果API提供者承擔起更大的責任，針對他們的使用者進行測試，並明確宣傳預計會有哪些型別的變化，我們就有可能在更大的範圍內建立更高模擬的依賴網路。

## TL;DRs  內容提要

- Prefer source control problems to dependency management problems: if you can get more code from your organization to have better transparency and coordination, those are important simplifications.

- Adding a dependency isn」t free for a software engineering project, and the complexity in establishing an 『ongoing』 trust relationship is challenging. Importing dependencies into your organization needs to be done carefully, with an understanding of the ongoing support costs.

- A dependency is a contract: there is a give and take, and both providers and consumers have some rights and responsibilities in that contract. Providers should be clear about what they are trying to promise over time.

- SemVer is a lossy-compression shorthand estimate for 『How risky does a human think this change is?』 SemVer with a SAT-solver in a package manager takes those estimates and escalates them to function as absolutes. This can result in either overconstraint (dependency hell) or underconstraint (versions that should work together that don」t).

- By comparison, testing and CI provide actual evidence of whether a new set of versions work together.


- 更傾向於源控制問題，而不是依賴管理問題：如果你能從你的組織中獲得更多的程式碼，以便有更好的透明度和協調，這些都是重要的簡化。

- 對於一個軟體工程專案來說，增加一個依賴關係並不是免費的，建立一個 "持續"的信任關係的複雜性是具有挑戰性的。將依賴關係匯入你的組織需要謹慎行事，並瞭解持續支援的成本。

- 依賴關係是一個合同：有付出就有收穫，提供者和消費者在該合同中都有一些權利和責任。供應商應該清楚地瞭解他們在一段時間內試圖承諾什麼。

- SemVer是對 "人類認為這一變化的風險有多大 "的一種有損壓縮的速記估計。SemVer與軟體套件管理器中的SAT求解器一起，將這些估計值升級為絕對值。這可能會導致過度約束（依賴性地獄）或不足約束（應該一起工作的版本卻沒有）。

- 相比之下，測試和CI提供了一組新版本是否能一起工作的實際證據。

