
**CHAPTER 16**

# Version Control and Branch Management

# 第十六章 版本控制和分支管理

**Written by Titus Winters**

**Edited by Lisa Carey**

Perhaps no software engineering tool is quite as universally adopted throughout the industry as version control. One can hardly imagine any software organization larger than a few people that doesn」t rely on a formal Version Control System (VCS) to manage its source code and coordinate activities between engineers.

也許沒有一種軟體工程工具像版本控制工具那樣在整個行業中被廣泛採用。我們很難想象，任何超過幾個人的軟體組織不依靠正式的版本控制系統（VCS）來管理其源程式碼和協調工程師之間的活動。

In this chapter, we」re going to look at why the use of version control has become such an unambiguous norm in software engineering, and we describe the various possible approaches to version control and branch management, including how we do it at scale across all of Google. We」ll also examine the pros and cons of various approaches; although we believe everyone should use version control, some version control policies and processes might work better for your organization (or in general) than others. In particular, we find 『trunk-based development』 as popularized by DevOps[^1] (one repository, no dev branches) to be a particularly scalable policy approach, and we」ll provide some suggestions as to why that is.

在本章中，我們將瞭解為什麼版本控制工具的使用在軟體工程中已成為如此明確的規範，我們將描述版本控制和分支管理的各種可能方法，包括我們如何在整個谷歌範圍內大規模地使用。我們還將研究各種方法的優缺點；儘管我們認為每個人都應該使用版本控制，但某些版本控制策略和流程可能比其他策略和流程更適合你的組織（或總體而言）。特別是，我們發現由DevOps推廣的 "基於主幹的開發"（一個版本庫，沒有開發分支）是一種特別可擴充套件的策略方法，我們將提供一些建議來解釋為什麼會這樣。

> [^1]: The DevOps Research Association, which was acquired by Google between the first draft of this chapter and publication, has published extensively on this in the annual 『State of DevOps Report』 and the book Accelerate. As near as we can tell, it popularized the terminology trunk-based development.
>
> 1 DevOps研究協會，在本章初稿和出版之間被谷歌收購，在年度 "DevOps狀況報告 "和《加速》一書中廣泛發表了這方面的內容。據我們所知，它推廣了基於主幹的開發這一術語。

## What Is Version Control?  什麼是版本控制？

A VCS is a system that tracks revisions (versions) of files over time. A VCS maintains some metadata about the set of files being managed, and collectively a copy of the files and metadata is called a repository[^2] (repo for short). A VCS helps coordinate the activities of teams by allowing multiple developers to work on the same set of files simultaneously. Early VCSs did this by granting one person at a time the right to edit a file—that style of locking is enough to establish sequencing (an agreed-upon 『which is newer,』 an important feature of VCS). More advanced systems ensure that changes to a *collection* of files submitted at once are treated as a single unit (*atomicity* when a logical change touches multiple files). Systems like CVS (a popular VCS from the 90s) that didn」t have this atomicity for a commit were subject to corruption and lost changes. Ensuring atomicity removes the chance of previous changes being overwritten unintentionally, but requires tracking which version was last synced to—at commit time, the commit is rejected if any file in the commit has been modified at head since the last time the local developer synced. Especially in such a change-tracking VCS, a developer」s working copy of the managed files will therefore need metadata of its own. Depending on the design of the VCS, this copy of the repository can be a repository itself, or might contain a reduced amount of metadata—such a reduced copy is usually a 『client』 or 『workspace.』

VCS是一個跟蹤檔案隨時間變化的修訂（版本）的系統。VCS維護一些關於被管理的檔案集的元資料，檔案和元資料的副本統稱為版本庫（簡稱repo）。VCS透過允許多個開發者同時在同一組檔案上工作來幫助協調團隊的活動。早期的VCS是透過每次授予一個人編輯檔案的權利來實現的——這種鎖定方式足以建立順序（一種約定的『更新的』，VCS的一個重要屬性）。更高階的系統確保對一次送出的*檔案集合*的更改被視為單個單元（當邏輯更改涉及多個檔案時，原子性）。像CVS（90年代流行的VCS）這樣的系統，如果沒有這種送出的原子性，就會出現損壞和丟失更改。確保原子性消除了以前的更改被無意覆蓋的可能性，但需要跟蹤最後同步的版本——在送出時，如果送出中的任何檔案在本地開發者最後一次同步後被修改過，則送出將被拒絕。特別是在這樣一個變化跟蹤的VCS中，開發者管理著的檔案的工作副本因此需要有自己的元資料。根據VCS的設計，這個版本庫的副本可以是一個版本庫本身，也可以包含一個精簡的元資料——這樣一個精簡的副本通常是一個 "用戶端"或"工作區"。

This seems like a lot of complexity: why is a VCS necessary? What is it about this sort of tool that has allowed it to become one of the few nearly universal tools for software development and software engineering?

這似乎很複雜：為什麼需要一個VCS？是什麼讓這種工具成為為數不多的軟體開發和軟體工程幾乎通用的工具之一？

Imagine for a moment working without a VCS. For a (very) small group of distributed developers working on a project of limited scope without any understanding of version control, the simplest and lowest-infrastructure solution is to just pass copies of the project back and forth. This works best when edits are nonsimultaneous (people are working in different time zones, or at least with different working hours). If there」s any chance for people to not know which version is the most current, we immediately have an annoying problem: tracking which version is the most up to date. Anyone who has attempted to collaborate in a non-networked environment will likely recall the horrors of copying back-and-forth files named *Presentation v5 - final - redlines - Josh」s version v2*. And as we shall see, when there isn」t a single agreed-upon source of truth, collaboration becomes high friction and error prone.

想象一下，在沒有VCS的情況下工作。對於一個（非常）小的分散式開發人員小組，在一個範圍有限的專案上工作，而不瞭解版本控制，最簡單和最低要求的基礎設施解決方案是隻是來回傳遞專案的副本。這在非同步編輯時效果最好（人們在不同的時區工作，或至少在不同的工作時間）。如果有讓人們不知道哪個版本是最新的時刻，我們馬上就會有一個惱人的問題：追蹤哪個版本是最新的。任何試圖在非網路環境下進行協作的人都可能會想起來回複製名為*Presentation v5 - final - redlines - Josh's version v2*的檔案的恐怖。正如我們將看到的那樣， 當沒有一個統一的訊息來源時，合作就會變得阻力很大，容易出錯。

Introducing shared storage requires slightly more infrastructure (getting access to shared storage), but provides an easy and obvious solution. Coordinating work in a shared drive might suffice for a while with a small enough number of people but still requires out-of-band collaboration to avoid overwriting one another」s work. Further, working directly in that shared storage means that any development task that doesn」t keep the build working continuously will begin to impede everyone on the team—if I」m making a change to some part of this system at the same time that you kick off a build, your build won」t work. Obviously, this doesn」t scale well.

引入共享儲存需要稍多的基礎設施（獲得對共享儲存的訪問），但提供了一個簡單而顯著的解決方案。在一個共享儲存驅動器中協調工作，在人數足夠少的情況下可能已經足夠了，但仍然需要帶外協作以避免覆蓋彼此的工作。此外，直接在共享儲存中工作意味著任何不能保持建立持續工作的開發任務都會開始阻礙團隊中的每個人——如果我在你啟動建立的同時對這個系統的某些部分進行了修改，你的建立就無法工作。很明顯，這並不能很好地擴充套件。

In practice, lack of file locking and lack of merge tracking will inevitably lead to collisions and work being overwritten. Such a system is very likely to introduce out-of-band coordination to decide who is working on any given file. If that file-locking is encoded in software, we」ve begun reinventing an early-generation version control like RCS (among others). After you realize that granting write permissions a file at a time is too coarse grained and you begin wanting line-level tracking—we」re definitely reinventing version control. It seems nearly inevitable that we」ll want some structured mechanism to govern these collaborations. Because we seem to just be reinventing the wheel in this hypothetical, we might as well use an off-the-shelf tool.

在實踐中，缺乏檔案鎖和缺乏合併跟蹤將不可避免地導致衝突和工作被覆蓋。這樣一個系統很有可能引入帶外協調，以決定誰在任何給定的檔案上工作。如果這種檔案鎖定被編碼在軟體中，我們就開始重新發明像RCS（包括其他）這樣的早期版本控制。當你意識到一次授予一個檔案的寫入許可權過於粗放，而你開始需要行級跟蹤時，我們肯定在重新發明版本控制。似乎不可避免的是，我們將需要一些結構化的機制來管理這些合作。因為在這個假設中，我們似乎只是在重新發明車輪，我們不妨使用一個現成的工具。

> [^2]: Although the formal idea of what is and is not a repository changes a bit depending on your choice of VCS, and the terminology will vary.
>
> 2 雖然什麼是和什麼不是版本庫的正式概念會因你選擇的VCS而有些變化，術語也會有所不同。

### Why Is Version Control Important?  為什麼版本控制很重要？

While version control is practically ubiquitous now, this was not always the case. The very first VCSs date back to the 1970s (SCCS) and 1980s (RCS)—many years later than the first references to software engineering as a distinct discipline. Teams participated in 『the [multiperson development of multiversion software](https://arxiv.org/pdf/1805.02742.pdf)』 before the industry had any formal notion of version control. Version control evolved as a response to the novel challenges of digital collaboration. It took decades of evolution and dissemination for reliable, consistent use of version control to evolve into the norm that it is today.[^3] So how did it become so important, and, given that it seems like a self-evident solution, why might anyone resist the idea of VCS?

雖然現在版本控制幾乎無處不在，但情況並非總是如此。最早的VCS可以追溯到20世紀70年代（SCCS）和80年代（RCS）——比首次將軟體工程作為一門獨立學科的時間晚了許多年。在業界有任何正式的版本控制概念之前，團隊就參與了"[多版本軟體的多人開發](https://arxiv.org/pdf/1805.02742.pdf)"。版本控制是為了應對數字協作的新挑戰而發展起來的。經過幾十年的演變和傳播，版本控制的可靠、一致的使用才演變成今天的規範。 那麼，它是如何變得如此重要的呢？鑑於它似乎是一個不言而喻的解決方案，為什麼會有人抵制VCS的想法呢？

Recall that software engineering is programming integrated over time; we」re drawing a distinction (in dimensionality) between the instantaneous production of source code and the act of maintaining that product over time. That basic distinction goes a long way to explaining the importance of, and hesitation toward, VCS: at the most fundamental level, version control is the engineer」s primary tool for managing the interplay between raw source and time. We can conceptualize VCS as a way to extend a standard filesystem. A filesystem is a mapping from filename to contents. A VCS extends that to provide a mapping from (filename, time) to contents, along with the metadata necessary to track last sync points and audit history. Version control makes the consideration of time an explicit part of the operation: unnecessary in a program‐ming task, critical in a software engineering task. In most cases, a VCS also allows for an extra input to that mapping (a branch name) to allow for parallel mappings; thus:

回顧一下，軟體工程是隨著時間的推移而整合的程式設計；我們在原始碼的即時生產和隨著時間的推移維護該產品的行為之間（在維度上）進行了區分。這一基本區別在很大程度上解釋了VCS的重要性和對VCS的疑慮：在最基本的層面上，版本控制是工程師管理原始源和時間之間相互作用的主要工具。我們可以將VCS概念化為一種擴充套件標準檔案系統的方式。檔案系統是一個從檔案名到內容的對映。VCS擴充套件了它，提供了從（檔案名，時間）到內容的對映，以及跟蹤最後同步點和審計歷史所需的元資料。版本控制把時間的操作的一個明確的部分：在編程任務中是不必要的，在軟體工程任務中是關鍵的。在大多數情況下，VCS還允許對該對映有一個額外的輸入（一個分支名稱），以允許並行對映；因此：

```txt
VCS(filename, time, branch) => file contents
```

In the default usage, that branch input will have a commonly understood default: we call that 『head,』 『default,』 or 『trunk』 to denote main branch.

在預設的用法中，該分支輸入將有一個普遍理解的預設值：我們稱之為『head』、『default』或『trunk』來表示主分支

The (minor) remaining hesitation toward consistent use of version control comes almost directly from conflating programming and software engineering—we teach programming, we train programmers, we interview for jobs based on programming problems and techniques. It」s perfectly reasonable for a new hire, even at a place like Google, to have little or no experience with code that is worked on by more than one person or for more than a couple weeks. Given that experience and understanding of the problem, version control seems like an alien solution. Version control is solving a problem that our new hire hasn」t necessarily experienced: an 『undo,』 not for a single file but for an entire project, adding a lot of complexity for sometimes nonobvious benefits.

對持續使用版本控制的（微小的）剩餘疑慮幾乎直接來自於程式設計和軟體工程的融合——我們教程式設計，我們培訓工程師，我們根據程式設計問題和技術來面試工作。對於一個新員工來說，即使是在像谷歌這樣的地方，對於由一個以上的人或幾個星期以上的時間來處理的程式碼，幾乎沒有經驗，這是完全合理的。鑑於這種經驗和對問題的理解，版本控制似乎是一個陌生的解決方案。版本控制正在解決一個我們的新僱員不一定經歷過的問題：『撤銷』，不是針對單個檔案，而是針對整個專案，這增加了很多複雜性，有時並沒有帶來了明顯的好處。

In some software groups, the same result plays out when management views the job of the techies as 『software development』 (sit down and write code) rather than 『software engineering』 (produce code, keep it working and useful for some extended period). With a mental model of programming as the primary task and little understanding of the interplay between code and the passage of time, it」s easy to see something described as 『go back to a previous version to undo a mistake』 as a weird, high- overhead luxury.

在一些軟體團隊中，當管理層將技術人員的工作視為『軟體開發』（坐下來編寫程式碼）而不是『軟體工程』（生成程式碼，使其在較長時間內保持工作和有效）時，也會產生同樣的結果。在把程式設計作為主要任務的思維模式下，以及對程式碼和時間流逝之間的相互作用瞭解甚少的情況下，很容易把 "回傳到以前的版本以撤銷錯誤 "這樣的描述看作是一種奇怪的、高開銷的奢侈品。

In addition to allowing separate storage and reference to versions over time, version control helps us bridge the gap between single-developer and multideveloper processes. In practical terms, this is why version control is so critical to software engineering, because it allows us to scale up teams and organizations, even though we use it only infrequently as an 『undo』 button. Development is inherently a branch-and- merge process, both when coordinating between multiple developers or a single developer at different points in time. A VCS removes the question of 『which is more recent?』 Use of modern version control automates error-prone operations like tracking which set of changes have been applied. Version control is how we coordinate between multiple developers and/or multiple points in time.

除了允許隨著時間的推移單獨儲存和長期引用版本外，版本控制還幫助我們彌合單個開發人員和多個開發人員流程之間的差距。在實踐中，這就是為什麼版本控制對軟體工程如此關鍵，因為它允許我們擴大團隊和組織的規模，儘管我們只是不經常使用它作為一個 "撤銷"按鈕。**開發本質上是一個分支和合並的過程，無論是在多個開發人員之間還是在不同時間點的單個開發人員之間進行協調。**版本控制系統消除了 "哪個是最新的？"的問題。使用現代的版本控制可以將容易出錯的操作自動化，比如跟蹤哪一組修改已經被應用。版本控制是我們在多個開發者和/或多個時間點之間協調的方式。

Because VCS has become so thoroughly embedded in the process of software engineering, even legal and regulatory practices have caught up. VCS allows a formal record of every change to every line of code, which is increasingly necessary for satisfying audit requirements. When mixing between in-house development and appropriate use of third-party sources, VCS helps track provenance and origination for every line of code.

由於VCS已經完全融入到軟體工程的過程中，甚至連法律和監管實踐也迎頭趕上。VCS允許對每一行程式碼的每一次更改進行正式記錄，這對於滿足審計要求越來越必要。當混合使用內部開發和第三方資源時，VCS幫助跟蹤每行程式碼的出處和起源。

In addition to the technical and regulatory aspects of tracking source over time and handling sync/branch/merge operations, version control triggers some nontechnical changes in behavior. The ritual of committing to version control and producing a commit log is a trigger for a moment of reflection: what have you accomplished since your last commit? Is the source in a state that you」re happy with? The moment of introspection associated with committing, writing up a summary, and marking a task complete might have value on its own for many people. The start of the commit process is a perfect time to run through a checklist, run static analyses (see Chapter 20), check test coverage, run tests and dynamic analysis, and so on.

除了隨時間跟蹤源和處理同步/分支/合併操作的技術和法規方面外，版本控制還會觸發一些行為上的非技術性更改。送出到版本控制並生成送出日誌的慣例是引發思考的一刻：自上次送出以來，你完成了什麼？原始碼是否處於你滿意的狀態？對於許多人來說，與送出、撰寫總結和完成任務相關的內省時刻本身可能具有價值。送出過程的開始是執行檢查表、執行靜態分析（參見第20章）、檢查測試覆蓋率、執行測試和動態分析等的最佳時機。

Like any process, version control comes with some overhead: someone must configure and manage your version control system, and individual developers must use it. But make no mistake about it: these can almost always be pretty cheap. Anecdotally, most experienced software engineers will instinctively use version control for any project that lasts more than a day or two, even for a single-developer project. The consistency of that result argues that the trade-off in terms of value (including risk reduction) versus overhead must be a pretty easy one. But we」ve promised to acknowledge that context matters and to encourage engineering leaders to think for themselves. It is always worth considering alternatives, even on something as fundamental as version control.

與任何流程一樣，版本控制也會帶來一些開銷：必須有人設定和管理你的版本控制系統，並且每個開發人員都必須使用它。但別搞錯了：這些幾乎總是相當低成本的。有趣的是，大多數經驗豐富的軟體工程師會本能地對任何持續一到兩天以上的專案使用版本控制，即使是單個開發人員的專案。這一結果的一致性表明，在價值（包括風險降低）和管理費用方面的權衡必須非常容易。但我們要承認背景的重要性，並鼓勵工程負責人獨立思考。即使在像版本控制這樣的基本問題上，也總是值得考慮其他選擇。

In truth, it」s difficult to envision any task that can be considered modern software engineering that doesn」t immediately adopt a VCS. Given that you understand the value and need for version control, you are likely now asking what type of version control you need.

事實上，很難設想任何可以被認為是現代軟體工程的任務不立即採用VCS。鑑於你瞭解版本控制的價值和需要，你現在可能會問你需要什麼型別的版本控制。

> [^3]: Indeed, I」ve given several public talks that use 『adoption of version control』 as the canonical example of how the norms of software engineering can and do evolve over time. In my experience, in the 1990s, version control was pretty well understood as a best practice but not universally followed. In the early 2000s, it was still common to encounter professional groups that didn」t use it. Today, the use of tools like Git seems ubiquitous even among college students working on personal projects. Some of this rise in adoption is likely due to better user experience in the tools (nobody wants to go back to RCS), but the role of experience and changing norms is significant.
>
> 3 事實上，我曾多次公開演講，以 "版本控制的採用 "為例，說明軟體工程的規範是如何隨著時間的推移而演變的。根據我的經驗，在20世紀90年代，版本控制被理解為一種最佳實踐，但沒有得到普遍遵守。在21世紀初，不使用版本控制的專業團體仍然很常見。今天，即使在從事個人專案的大學生中，像Git這樣的工具的使用似乎也是無處不在的。這種採用率的上升可能是由於在工具中更好的使用者體驗（沒有人願意回到RCS），但經驗和不斷變化的規範的作用也很重要。

### Centralized VCS Versus Distributed VCS  集中式VCS與分散式VCS

At the most simplistic level, all modern VCSs are equivalent to one another: so long as your system has a notion of atomically committing changes to a batch of files, everything else is just UI. You could build the same general semantics (not workflow) of any modern VCS out of another one and a pile of simple shell scripts. Thus, arguing about which VCS is 『better』 is primarily a matter of user experience—the core functionality is the same, the differences come in user experience, naming, edge-case features, and performance. Choosing a VCS is like choosing a filesystem format: when choosing among a modern-enough format, the differences are fairly minor, and the more important question by far is the content you fill that system with and the way you *use* it. However, major architectural differences in VCSs can make configuration, policy, and scaling decisions easier or more difficult, so it」s important to be aware of the big architectural differences, chiefly the decision between centralized or decentralized.

在最簡單的層面上，所有現代VCS都是一樣的：只要你的系統有一個將更改以原子方式送出給一批檔案的概念，其他一切都只是UI不同。你可以用另一個VCS和一堆簡單的shell指令碼建立任何現代VCS的通用語義（而不是工作流）。因此，討論哪些VCS『更好』主要是使用者體驗的問題。核心功能是相同的，不同之處在於使用者體驗、命名、邊緣案例功能和效能。選擇一個VCS就像選擇一個檔案系統格式：在一個足夠現代的格式中進行選擇時，差異是相當小的，到目前為止，更重要的問題是你在該系統中填充的內容以及你使用它的方式。然而，VCS中的主要架構差異可能會使設定、策略和擴充套件決策變得更容易或更困難，因此重要的是要意識到巨大的架構差異，主要是集中式和分散式之間的決策。

#### Centralized VCS  集中式VCS

In centralized VCS implementations, the model is one of a single central repository (likely stored on some shared compute resource for your organization). Although a developer can have files checked out and accessible on their local workstation, operations that interact on the version control status of those files need to be communicated to the central server (adding files, syncing, updating existing files, etc.). Any code that is committed by a developer is committed into that central repository. The first VCS implementations were all centralized VCSs.

在集中式的VCS實現中，模型是一個單一的中央儲存庫（可能儲存在你的組織的一些共享計算資源上）。儘管開發者可以在他們的本地工作站上籤出和訪問檔案，但與這些檔案的版本控制狀態互動的操作需要與中央伺服器通訊（新增檔案、同步、更新現有檔案，等等）。任何由開發者送出的程式碼都會被送出到中央儲存庫。第一批VCS的實現都是集中式VCS。

Going back to the 1970s and early 1980s, we see that the earliest of these VCSs, such as RCS, focused on locking and preventing multiple simultaneous edits. You could copy the contents of a repository, but if you wanted to edit a file, you might need to acquire a lock, enforced by the VCS, to ensure that only you are making edits. When you」ve completed an edit, you release the lock. The model worked fine when any given change was a quick thing, or if there was rarely more than one person that wanted the lock for a file at any given time. Small edits like tweaking config files worked OK, as did working on a small team that either kept disjointed working hours or that rarely worked on overlapping files for extended periods. This sort of simplistic locking has inherent problems with scale: it can work fine for a few people, but has the potential to fall apart with larger groups if any of those locks become contended.[^4]

回溯到20世紀70年代和80年代初，我們看到最早的這些VCS，如RCS，側重於鎖定和防止多個同時編輯。你可以複製版本庫的內容，但如果你想編輯一個檔案，你可能需要獲得一個鎖，由VCS強制執行的鎖，以確保只有你在進行編輯。當你完成了一個編輯，你就可以釋放鎖。當任何給定的變化是一個很快的事情，或者在任何給定的時間內很少有超過一個人想要鎖定一個檔案時，這種模式工作得很好。像調整設定檔案這樣的小的編輯工作是可以的，就像在一個小團隊中工作一樣，這個團隊要麼保持不連貫的工作時間，要麼很少長時間處理重疊的檔案。這種簡單化的鎖定在規模上有固有的問題：對幾個人來說，它可以很好地工作，但如果這些鎖中的任何一個被爭奪，就有可能在較大的群體中崩潰。

As a response to this scaling problem, the VCSs that were popular through the 90s and early 2000s operated at a higher level. These more modern centralized VCSs avoid the exclusive locking but track which changes you」ve synced, requiring your edit to be based on the most-current version of every file in your commit. CVS wrapped and refined RCS by (mostly) operating on batches of files at a time and allowing multiple developers to check out a file at the same time: so long as your base version contained all of the changes in the repository, you」re allowed to commit. Subversion advanced further by providing true atomicity for commits, version tracking, and better tracking for unusual operations (renames, use of symbolic links, etc.). The centralized repository/checked-out client model continues today within Subversion as well as most commercial VCSs.

作為對這一規模問題的回應，在90年代和21世紀初流行的VCS在更高水平上執行。這些更現代化的集中式VCS避免了獨佔鎖定，但會跟蹤你已同步的更改，要求你的編輯基於送出中每個檔案的最新版本。CVS透過（主要是）一次操作一批檔案並允許多個開發人員同時簽出一個檔案來包裝和細化RCS：只要你的基礎版本包含儲存庫中的所有更改，你就可以送出。Subversion透過提供真正的送出原子性、版本跟蹤和對不尋常操作（重新命名、使用符號連結等）的更好跟蹤而進一步發展。集中式版本庫/檢出用戶端的模式在Subversion以及大多數商業VCS中延續至今。

> [^4]: Anecdote: To illustrate this, I looked for information on what pending/unsubmitted edits Googlers had outstanding for a semipopular file in my most recent project. At the time of this writing, 27 changes are pending, 12 from people on my team, 5 from people on related teams, and 10 from engineers I」ve never met. This is basically working as expected. Technical systems or policies that require out-of-band coordination certainly don」t scale to 24/7 software engineering in distributed locations.
>
> 4   軼事：為了說明這一點，我尋找了谷歌在我最近的專案中對一個半流行的檔案所做的未送出/未送出編輯的訊息。在撰寫本文時，有27項變更尚未完成，其中12項來自我的團隊，5項來自相關團隊，10項來自我從未見過的工程師。這基本上按照預期工作。需要帶外協調的技術系統或策略當然不能擴充套件到分散式位置的全天候軟體工程。

#### Distributed VCS 分散式VCS

Starting in the mid-2000s, many popular VCSs followed the Distributed Version Control System (DVCS) paradigm, seen in systems like Git and Mercurial. The primary conceptual difference between DVCS and more traditional centralized VCS (Subversion, CVS) is the question: 『Where can you commit?』 or perhaps, 『Which copies of these files count as a repository?』

從2000年中期開始，許多流行的VCS遵循分散式版本控制系統（DVCS）的正規化，在Git和Mercurial等系統中看到。DVCS和更多傳統的集中式VCS（Subversion，CVS）之間的主要概念差異在於問題："你可以在哪裡送出？"或者說，"這些檔案的哪些副本算作一個儲存庫？"

A DVCS world does not enforce the constraint of a central repository: if you have a copy (clone, fork) of the repository, you have a repository that you can commit to as well as all of the metadata necessary to query for information about things like revision history. A standard workflow is to clone some existing repository, make some edits, commit them locally, and then push some set of commits to another repository, which may or may not be the original source of the clone. Any notion of centrality is purely conceptual, a matter of policy, not fundamental to the technology or the underlying protocols.

DVCS世界不強制執行中央儲存庫的約束：如果你有儲存庫的副本（克隆、分叉），那麼你就有一個可以送出的儲存庫以及查詢有關修訂歷史等訊息所需的所有元資料。標準工作流是克隆一些現有儲存庫，進行一些編輯，在本地送出，然後將一組送出推送到另一個儲存庫，該儲存庫可能是克隆的原始源，也可能不是克隆的原始源。任何關於中心性的概念都純粹是概念性的，是一個策略問題，而不是技術或底層協定的根本。

The DVCS model allows for better offline operation and collaboration without inherently declaring one particular repository to be the source of truth. One repository isn」t necessary 『ahead』 or 『behind』 because changes aren」t inherently projected into a linear timeline. However, considering common *usage*, both the centralized and DVCS models are largely interchangeable: whereas a centralized VCS provides a clearly defined central repository through technology, most DVCS ecosystems define a central repository for a project as a matter of policy. That is, most DVCS projects are built around one conceptual source of truth (a particular repository on GitHub, for instance). DVCS models tend to assume a more distributed use case and have found particularly strong adoption in the open source world.

DVCS模型允許更好的離線操作和協作，而無需預先宣告某個特定儲存庫是訊息源。一個儲存庫不必『領先』或『落後』，因為更改不會固有地投射到線性時間線中。然而，考慮到通用性，集中式和DVCS模型在很大程度上是可互換的：集中式VCS透過技術提供了一個明確定義的中央儲存庫，而大多數DVCS生態系統將專案的中央儲存庫定義為一個策略問題。也就是說，大多數DVCS專案都是圍繞一個訊息源的概念（例如GitHub上的特定儲存庫）建立的。DVCS模型傾向於假設一個更分散式的用例，並且在開源世界中得到了特別強烈的採用。

Generally speaking, the dominant source control system today is Git, which implements DVCS.[^5] When in doubt, use that—there」s some value in doing what everyone else does. If your use cases are expected to be unusual, gather some data and evaluate the trade-offs.

一般來說，今天占主導地位的原始碼控制系統是Git，它實現了DVCS。當有疑問時，使用它——做別人做的事是有價值的。如果你的用例預期不尋常，請收集一些資料並評估權衡。

Google has a complex relationship with DVCS: our main repository is based on a (massive) custom in-house centralized VCS. There are periodic attempts to integrate more standard external options and to match the workflow that our engineers (especially Nooglers) have come to expect from external development. Unfortunately, those attempts to move toward more common tools like Git have been stymied by the sheer size of the codebase and userbase, to say nothing of Hyrum」s Law effects tying us to a particular VCS and interface for that VCS.[^6] This is perhaps not surprising: most existing tools don」t scale well with 50,000 engineers and tens of millions of commits.[^7] The DVCS model, which often (but not always) includes transmission of history and metadata, requires a lot of data to spin up a repository to work out of.

谷歌與DVCS有著複雜的關係：我們的主要資源庫是基於一個（巨大的）自定義的內部集中式VCS。我們定期嘗試整合更多標準的外部選項，並與我們的工程師（尤其是Nooglers）所期望的外部開發的工作流程相對應。不幸的是，由於程式碼庫和使用者群的巨大規模，以及海勒姆定律的影響，這些向Git這樣的通用工具發展的嘗試受到了阻礙，更不用說將我們束縛在一個特定的VCS和VCS的介面上了。這也許並不奇怪：大多數現有的工具在面對5萬名工程師和數千萬的送出時都不能很好地擴充套件。DVCS模型，通常（但不總是）包括歷史和元資料的傳輸，需要大量資料來加速儲存庫的執行。

In our workflow, centrality and in-the-cloud storage for the codebase seem to be critical to scaling. The DVCS model is built around the idea of downloading the entire codebase and having access to it locally. In practice, over time and as your organization scales up, any given developer is going to operate on a relatively smaller percentage of the files in a repository, and a small fraction of the versions of those files. As we grow (in file count and engineer count), that transmission becomes almost entirely waste. The only need for locality for most files occurs when building, but distributed (and reproducible) build systems seem to scale better for that task as well (see Chapter 18).

在我們的工作流程中，程式碼庫的中心化和雲儲存似乎對擴充套件至關重要。DVCS模型是圍繞下載整個程式碼庫並在本地訪問它的思想建立的。在實踐中，隨著時間的推移和組織規模的擴大，任何給定的開發人員都會在相對較小比例的檔案庫中進行操作，而且這些檔案的版本也只佔一小部分。隨著我們的增長（在檔案數和工程師數方面），這種傳輸幾乎完全變成了浪費。大多數檔案在建立時只需要本地區域性檔案，但分散式（和可複製的）建立系統似乎也能更好地擴充套件該任務（參見第18章）。

> [^5]: Stack Overflow Developer Survey Results, 2018.
>
> 5 Stack Overflow開發者調查結果，2018年。
>
> [^6]: Monotonically increasing version numbers, rather than commit hashes, are particularly troublesome. Many systems and scripts have grown up in the Google developer ecosystem that assume that the numeric ordering of commits is the same as the temporal order—undoing those hidden dependencies is difficult.
>
> 6 單調增加的版本號，而不是送出雜湊值，是特別麻煩的。許多系統和指令碼已經在谷歌開發者生態系統中成長起來，它們假定送出的數字順序與時間順序相同--消除這些隱藏的依賴關係是很困難的。
>
> [^7]: For that matter, as of the publication of the Monorepo paper, the repository itself had something like 86 TB of data and metadata, ignoring release branches. Fitting that onto a developer workstation directly would be… challenging.
>
> 7 就這一點而言，截至Monorepo論文發表時，倉庫本身有大約86TB的資料和元資料，不包括發布分支。將其直接裝入開發者的工作站將是......挑戰。

### Source of Truth 訊息源

Centralized VCSs (Subversion, CVS, Perforce, etc.) bake the source-of-truth notion into the very design of the system: whatever is most recently committed at trunk is the current version. When a developer goes to check out the project, by default that trunk version is what they will be presented with. Your changes are 『done』 when they have been recommitted on top of that version.

集中式VCS（Subversion、CVS、Perforce等）將訊息源的概念融入到系統的設計中：最近送出到主幹的就是當前的版本。當一個開發者去檢查專案時，預設情況下，他們將看到的是主幹版本。當你的修改被重新送出到該版本上時，你的修改就 "完成"了。

However, unlike centralized VCS, there is no *inherent* notion of which copy of the distributed repository is the single source of truth in DVCS systems. In theory, it」s possible to pass around commit tags and PRs with no centralization or coordination, allowing disparate branches of development to propagate unchecked, and thus risking a conceptual return to the world of *Presentation v5 - final - redlines - Josh」s version* *v2*. Because of this, DVCS requires more explicit policy and norms than a centralized VCS does.

然而，與集中式 VCS 不同，在 DVCS 系統中，並不存在哪個分散式版本庫的副本是單訊息源的*固有概念*。理論上，在沒有集中化或協調的情況下，送出標籤和PR的傳遞是可能的，允許不同的開發分支不受檢查地傳播，從而有可能在概念上回到*Presentation v5 - final - redlines - Josh's version v2*的世界。正因為如此，DVCS比集中式VCS需要更明確的策略和規範。

Well-managed projects using DVCS declare one specific branch in one specific repository to be the source of truth and thus avoid the more chaotic possibilities. We see this in practice with the spread of hosted DVCS solutions like GitHub or GitLab— users can clone and fork the repository for a project, but there is still a single primary repository: things are 『done』 when they are in the trunk branch on that repository.

使用DVCS的管理良好的專案宣佈一個特定的分支在一個特定的儲存庫中是訊息源，從而避免了更多混亂的可能性。在實踐中，我們看到GitHub或GitLab等託管DVCS解決方案的普及--使用者可以克隆和分叉一個專案的倉庫，但仍有一個單一的主倉庫：當事情出現在該倉庫的主幹分支時，就已經"完成"了。

It isn」t an accident that centralization and Source of Truth has crept back into the usage even in a DVCS world. To help illustrate just how important this Source of Truth idea is, let」s imagine what happens when we don」t have a clear source of truth.

即使在DVCS的世界裡，集中化和訊息源已經悄悄地回到了人們的使用中，這並不是一個偶然。為了說明 "訊息源 "這個概念有多重要，讓我們想象一下，當我們沒有明確的訊息源時會發生什麼。

#### Scenario: no clear source of truth  情景：沒有明確的訊息源

Imagine that your team adheres to the DVCS philosophy enough to avoid defining a specific branch+repository as the ultimate source of truth.

想象一下，你的團隊堅持DVCS的理念，足以避免將特定的分支+版本庫定義為最終的訊息源。

In some respects, this is reminiscent of the *Presentation v5 - final - redlines - Josh」s version v2* model—after you pull from a teammate」s repository, it isn」t necessarily clear which changes are present and which are not. In some respects, it」s better than that because the DVCS model tracks the merging of individual patches at a much finer granularity than those ad hoc naming schemes, but there」s a difference between the DVCS knowing *which* changes are incorporated and every engineer being sure they have *all* the past/relevant changes represented.

在某些方面，這讓人想起*Presentation v5 - final - redlines - Josh's version v2*的模式——當你從隊友的版本庫中提取後，並不一定清楚哪些改動是存在的，哪些是不存在的。在某些方面，它比這更好，因為DVCS模型在更細的粒度上跟蹤單個補丁的合併，而不是那些臨時的命名方案，但DVCS知道*哪些*變化被納入，和每個工程師確保他們已經表示了*所有*過去/相關的更改，這兩者之間存在差異。。

Consider what it takes to ensure that a release build includes all of the features that have been developed by each developer for the past few weeks. What (noncentralized, scalable) mechanisms are there to do that? Can we design policies that are fundamentally better than having everyone sign off? Are there any that require only sublinear human effort as the team scales up? Is that going to continue working as the number of developers on the team scales up? As far as we can see: probably not. Without a central Source of Truth, someone is going to keep a list of which features are potentially ready to be included in the next release. Eventually that bookkeeping is reproducing the model of having a centralized Source of Truth.

考慮一下如何確保一個發布版本包括每個開發人員在過去幾周內開發的所有功能。有什麼（非集中的、可擴充套件的）機制可以做到這一點？我們能不能設計出從根本上比讓每個人簽字更好的策略？是否有任何隨著團隊規模的擴大只需要亞線性的人力努力？隨著團隊中開發人員數量的增加，這是否會繼續發揮作用？就我們所見：可能不會。如果沒有一個核心的 "訊息源"，就會有人記下哪些功能有可能被納入下一個版本的清單。最終，這種記賬方式正在重現擁有一個集中式訊息源的模式。

Further imagine: when a new developer joins the team, where do they get a fresh, known-good copy of the code?

進一步想象：當一個新的開發人員加入團隊時，他們從哪裡得到一個最新的、已知的好的程式碼副本？

DVCS enables a lot of great workflows and interesting usage models. But if you」re concerned with finding a system that requires sublinear human effort to manage as the team grows, it」s pretty important to have one repository (and one branch) actually defined to be the ultimate source of truth.

DVCS實現了很多出色的工作流程和有趣的使用模式。但如果你關心的是找到一個系統，隨著團隊的成長，需要亞線性的人力來管理，那麼將一個儲存庫（和一個分支）實際定義為最終的訊息源是相當重要的。

There is some relativity in that Source of Truth. That is, for a given project, that Source of Truth might be different for a different organization. This caveat is important: it」s reasonable for engineers at Google or RedHat to have different Sources of Truth for Linux Kernel patches, still different than Linus (the Linux Kernel maintainer) himself would. DVCS works fine when organizations and their Sources of Truth are hierarchical (and invisible to those outside the organization)—that is perhaps the most practically useful effect of the DVCS model. A RedHat engineer can commit to the local Source of Truth repository, and changes can be pushed from there upstream periodically, while Linus has a completely different notion of what is the Source of Truth. So long as there is no choice or uncertainty as to where a change should be pushed, we can avoid a large class of chaotic scaling problems in the DVCS model.

訊息源具有某種相對性。也就是說，對於一個特定的專案，訊息源對於不同的組織可能是不同的。這一點很重要：谷歌或RedHat的工程師對Linux核心補丁有不同的訊息源是合理的，這與Linus（Linux核心維護者）自己的訊息源還是不同的。當組織和他們的訊息源是分層的（對組織外的人來說是不可見的），DVCS就能很好地工作——這也許是DVCS模型最實際的作用。一個RedHat的工程師可以送出到本地訊息源倉庫，並且可以定期從那裡向上遊推送變化，而Linus對什麼是訊息源有完全不同的概念。只要沒有選擇或不確定一個變化應該被推到哪裡，我們就可以避免DVCS模型中的一大類混亂的擴充套件問題。

In all of this thinking, we」re assigning special significance to the trunk branch. But of course, 『trunk』 in your VCS is only the technology default, and an organization can choose different policies on top of that. Perhaps the default branch has been abandoned and all work actually happens on some custom development branch—other than needing to provide a branch name in more operations, there」s nothing inherently broken in that approach; it」s just nonstandard. There」s an (oft-unspoken) truth when discussing version control: the technology is only one part of it for any given organization; there is almost always an equal amount of policy and usage convention on top of that.

在所有這些想法中，我們為主幹分支賦予了特殊的意義。但當然，VCS中的 "主幹"只是技術預設，一個組織可以在此基礎上選擇不同的策略。也許預設的分支已經被放棄了，所有的工作實際上都發生在某個自定義的開發分支上——除了需要在更多操作中提供分支名稱之外，這種方法沒有任何內在的缺陷；它只是非標準的。在討論版本控制時，有一個（經常不說的）事實：對於任何特定的組織來說，技術只是其中的一部分；幾乎總是有同等數量的策略和使用約定在上面。

No topic in version control has more policy and convention than the discussion of how to use and manage branches. We look at branch management in more detail in the next section.

版本控制中沒有一個主題比關於如何使用和管理分支的討論更具策略和約定。我們將在下一節更詳細地介紹分支管理。

### Version Control Versus Dependency Management 版本控制與依賴管理

There」s a lot of conceptual similarity between discussions of version control policies and dependency management (see [Chapter 21](#_bookmark1845)). The differences are primarily in two forms: VCS policies are largely about how you manage your own code, and are usually much finer grained. Dependency management is more challenging because we primarily focus on projects managed and controlled by other organizations, at a higher granularity, and these situations mean that you don」t have perfect control. We」ll discuss a lot more of these high-level issues later in the book.

關於版本控制策略和依賴管理的討論在概念上有很多相似之處（見第21章）。差異主要體現在兩種形式上。VCS策略主要是關於你如何管理你自己的程式碼，而且通常是更細的粒度。依賴管理更具挑戰性，因為我們主要關注由其他組織管理和控制的專案，顆粒度更高，這些情況意味著你沒有完美的控制。我們將在本書後面討論更多的這些高階問題。

## Branch Management  分支管理

Being able to track different revisions in version control opens up a variety of different approaches for how to manage those different versions. Collectively, these different approaches fall under the term *branch management*, in contrast to a single 『trunk.』

能夠在版本控制中跟蹤不同的修訂版，為如何管理這些不同的版本提供了各種不同的方法。總的來說，這些不同的方法屬於*分支管理*，與單一的 "主幹 "形成對比。

### Work in Progress Is Akin to a Branch  正在進行的工作類似於一個分支

Any discussion that an organization has about branch management policies ought to at least acknowledge that every piece of work-in-progress in the organization is equivalent to a branch. This is more explicitly the case with a DVCS in which developers are more likely to make numerous local staging commits before pushing back to the upstream Source of Truth. This is still true of centralized VCSs: uncommitted local changes aren」t conceptually different than committed changes on a branch, other than potentially being more difficult to find and diff against. Some centralized systems even make this explicit. For example, when using Perforce, every change is given two revision numbers: one indicating the implicit branch point where the change was created, and one indicating where it was recommitted, as illustrated in [Figure 16-1](#_bookmark1418). Perforce users can query to see who has outstanding changes to a given file, inspect the pending changes in other users」 uncommitted changes, and more.

組織對分支機構管理策略的任何討論都應該至少承認組織中正在進行的每一項工作都相當於一個分支。這一點在DVCS中更為明顯，因為在DVCS中，開發者更有可能在推送回上游訊息源之前進行大量本地暫存送出。集中式VCS仍然如此：未送出的本地更改在概念上與分支上送出的更改沒有區別，只是可能更難發現和區分。一些集中式系統甚至明確了這一點。例如，當使用Perforce時，每個更改都會有兩個修訂號：一個表示建立更改的隱含分支點，另一個表示重新送出更改的位置，如圖16-1所示。Perforce使用者可以查詢檢視誰對給定檔案有未完成的更改，檢查其他使用者未送出更改中的未決更改，等等。

![Figure 16-1. Two revision numbers in Perforce](./images/Figure%2016-1.png)

*Figure 16-1. Two revision numbers in Perforce*  *圖 16-1. Perforce中的兩個修訂號*

This 『uncommitted work is akin to a branch』 idea is particularly relevant when thinking about refactoring tasks. Imagine a developer being told, 『Go rename Widget to OldWidget.』 Depending on an organization」s branch management policies and understanding, what counts as a branch, and which branches matter, this could have several interpretations:

- Rename Widget on the trunk branch in the Source of Truth repository
- Rename Widget on all branches in the Source of Truth repository
- Rename Widget on all branches in the Source of Truth repository, and find all devs with outstanding changes to files that reference Widget

這個 "未送出的工作類似於分支 "的想法在思考重構任務時特別重要。想象一下，一個開發者被告知，"將Widget重新命名為OldWidget"。根據組織的分支管理策略和理解，什麼是分支，以及哪個分支重要，這可能有幾種解釋：

- 在訊息源版本庫的主幹分支上重新命名Widget
- 在訊息源版本庫中的所有分支上重新命名Widget
- 在訊息源版本庫的所有分支上重新命名Widget，並找到所有對引用Widget的檔案有未完成修改的開發者。

If we were to speculate, attempting to support that 『rename this everywhere, even in outstanding changes』 use case is part of why commercial centralized VCSs tend to track things like 『which engineers have this file open for editing?』 (We don」t think this is a scalable way to *perform* a refactoring task, but we understand the point of view.)

如果我們猜測，試圖支援『到處重新命名，即使在未完成的更改中』用例是為什麼商業集中式VCS傾向於跟蹤『哪些工程師開啟此檔案進行編輯？』（我們不認為這是執行重構任務的可擴充套件方式，但我們理解這個觀點。）

### Dev Branches  開發分支

In the age before consistent unit testing (see Chapter 11), when the introduction of any given change had a high risk of regressing functionality elsewhere in the system, it made sense to treat *trunk* specially. 『We don」t commit to trunk,』 your Tech Lead might say, 『until new changes have gone through a full round of testing. Our team uses feature-specific development branches instead.』

在沒有一致的單元測試的時代（見第11章），當任何給定的更改的引入都有很大的風險會使系統中其他地方的功能回滾時，特別對待*trunk*是有意義的。"我們不會向主幹送出，"你的技術負責人可能會說，"在新的變更透過一輪測試之前，我們不會合併搭配主幹。我們的團隊使用特定於功能的開發分支。"

A development branch (usually 『dev branch』) is a halfway point between 『this is done but not committed』 and 『this is what new work is based on.』 The problem that these are attempting to solve (instability of the product) is a legitimate one—but one that we have found to be solved far better with more extensive use of tests, Continuous Integration (CI) (see Chapter 23), and quality enforcement practices like thorough code review.

開發分支（通常是 "dev branch"）是介於 "這個已經完成但未送出 "和 "這個是新工作的基礎 "之間的中間點。這些試圖解決的問題（產品的不穩定性）是一個合理的問題，但我們發現透過更廣泛地使用測試、持續整合（CI）（見第23章）和徹底的程式碼審查等質量執行實踐可以更好地解決這個問題。

We believe that a version control policy that makes extensive use of dev branches as a means toward product stability is inherently misguided. The same set of commits are going to be merged to trunk eventually. Small merges are easier than big ones. Merges done by the engineer who authored those changes are easier than batching unrelated changes and merging later (which will happen eventually if a team is sharing a dev branch). If presubmit testing on the merge reveals any new problems, the same argument applies: it」s easier to determine whose changes are responsible for a regression if there is only one engineer involved. Merging a large dev branch implies that more changes are happening in that test run, making failures more difficult to isolate. Triaging and root-causing the problem is difficult; fixing it is even worse.

我們認為，大量使用開發分支作為產品穩定性手段的版本控制策略本身上是錯誤的。同一組送出最終將合併到主幹中。小的合併比大的合併容易。由編寫這些更改的工程師進行的合併比把不相關的修改分批合併要容易（如果團隊共享開發分支，最終會發生這種情況）。如果對合並進行的預送出測試發現了任何新問題，同樣的論點也適用：如果只有一名工程師參與，則更容易確定誰的更改導致了迴歸。合併一個大型開發分支意味著在該測試執行中會發生更多的更改，從而使故障更難隔離。處理和根除問題是困難的，而修復問題就更難了。

Beyond the lack of expertise and inherent problems in merging a single branch, there are significant scaling risks when relying on dev branches. This is a very common productivity drain for a software organization. When there are multiple branches being developed in isolation for long periods, coordinating merge operations becomes significantly more expensive (and possibly riskier) than they would be with trunk-based development.

除了在合併單個分支時缺乏專業知識和固有問題之外，依賴開發分支時還存在重大的擴充套件風險。對於軟體組織來說，這是一種非常常見的生產力損失。當有多個分支長期獨立開發時，協調合併操作會比基於主幹的開發成本更高（可能更高）。

#### How did we become addicted to dev branches?  我們是如何沉迷於開發分支的？

It」s easy to see how organizations fall into this trap: they see, 『Merging this long-lived development branch reduced stability』 and conclude, 『Branch merges are risky.』 Rather than solve that with 『Better testing』 and 『Don」t use branch-based development strategies,』 they focus on slowing down and coordinating the symptom: the branch merges. Teams begin developing new branches based on other in-flight branches. Teams working on a long-lived dev branch might or might not regularly have that branch synched with the main development branch. As the organization scales up, the number of development branches grows as well, and the more effort is placed on coordinating that branch merge strategy. Increasing effort is thrown at coordination of branch merges—a task that inherently doesn」t scale. Some unlucky engineer becomes the Build Master/Merge Coordinator/Content Management Engineer, focused on acting as the single point coordinator to merge all the disparate branches in the organization. Regularly scheduled meetings attempt to ensure that the organization has 『worked out the merge strategy for the week.』[^8] The teams that aren」t chosen to merge often need to re-sync and retest after each of these large merges.

很容易看出組織是如何落入這個陷阱的：他們看到，『合併這個長期存在的開發分支會降低穩定性』，並得出結論，『分支合併是有風險的。』而不是透過『更好的測試』和『不要使用基於分支的開發策略』來解決這個問題，只是專注於減緩和協調症狀：分支合併。團隊開始在其他正在執行的分支的基礎上開發新的分支。在一個長期存在的開發分支上工作的團隊可能會也可能不會定期讓該分支與主開發分支同步。隨著組織規模的擴大，開發分支的數量也在增加，在協調該分支合併策略上的努力也就越多。越來越多的精力投入到分支合併的協調上--這是一項本質上無法擴充套件的任務。一些不走運的工程師成為建立主管/合併協調人/內容管理工程師，專注於充當單點協調人，以合併組織中所有不同的分支。定期安排的會議試圖確保組織『制定了本週的合併策略』。未被選擇合併的團隊通常需要在每次大型合併後重新同步和測試。

All of that effort in merging and retesting is *pure overhead*. The alternative requires a different paradigm: trunk-based development, rely heavily on testing and CI, keep the build green, and disable incomplete/untested features at runtime. Everyone is responsible to sync to trunk and commit; no 『merge strategy』 meetings, no large/expensive merges. And, no heated discussions about which version of a library should be used—there can be only one. There must be a single Source of Truth. In the end, there will be a single revision used for a release: narrowing down to a single source of truth is just the 『shift left』 approach for identifying what is and is not being included.

所有這些合併和重新測試的努力都是*純粹的開銷*。替代方案需要一個不同的正規化：基於主幹的開發，嚴重依賴測試和CI，保持綠色建立，並在執行時停用不完整/未經測試的功能。每個人都有責任同步到主幹和送出；沒有 "合併策略 "會議，沒有大型/高成本的合併。而且，沒有關於應該使用哪個版本的函式庫的激烈討論--只能有一個。必須有一個單一的訊息源。最終，一個版本將使用一個單一的修訂版：縮小到一個單訊息源，這只是確定哪些是和哪些沒有被包括在內的『左移』方法。

> [^8]: Recent informal Twitter polling suggests about 25% of software engineers have been subjected to 『regularly scheduled』 merge strategy meetings.
>
> 8   最近的非正式推特民意調查顯示，大約25%的軟體工程師參加了『定期』的合併策略會議。

### Release Branches  發布分支

If the period between releases (or the release lifetime) for a product is longer than a few hours, it may be sensible to create a release branch that represents the exact code that went into the release build for your product. If any critical flaws are discovered between the actual release of that product into the wild and the next release cycle, fixes can be cherry-picked (a minimal, targeted merge) from trunk to your release branch.

如果某個產品的發布間隔（或發布生命週期）超過幾個小時，那麼建立一個發布分支來表示進入產品發布建立的確切程式碼可能是明智的。如果在該產品的實際發布和下一個發布週期之間發現了任何關鍵缺陷，那麼可以從主幹到你的發布分支進行修復（最小的、有針對性的合併）。

By comparison to dev branches, release branches are generally benign: it isn」t the technology of branches that is troublesome, it」s the usage. The primary difference between a dev branch and a release branch is the expected end state: a dev branch is expected to merge back to trunk, and could even be further branched by another team. A release branch is expected to be abandoned eventually.

與開發分支相比，發布分支通常是良性的：麻煩的不是分支的技術，而是用法。開發分支和發布分支的主要區別在於預期的最終狀態：開發分支預期會合併到主幹上，甚至可能會被另一個團隊進一步分支。而發布分支預計最終會被放棄。

In the highest-functioning technical organizations that Google」s DevOps Research and Assessment (DORA) organization has identified, release branches are practically nonexistent. Organizations that have achieved Continuous Deployment (CD)—the ability to release from trunk many times a day—likely tend to skip release branches: it」s much easier to simply add the fix and redeploy. Thus, cherry-picks and branches seem like unnecessary overhead. Obviously, this is more applicable to organizations that deploy digitally (such as web services and apps) than those that push any form of tangible release to customers; it is generally valuable to know exactly what has been pushed to customers.

在谷歌的DevOps研究和評估組織（DORA）所確定的最高效的技術組織中，發布分支實際上是不存在的。那些已經實現了持續部署（CD）的組織——每天多次從主幹發布的能力——很可能傾向於跳過發布分支：只需新增修復和重新部署就更容易了。因此，挑選（cherry-picks）和分支似乎是不必要的開銷。顯然，這更適用於以數字方式部署的組織（如網路服務和應用程式），而不是那些向客戶推送任何形式的有形發布的組織；通常，準確地瞭解向客戶推出的產品是很有價值的。

That same DORA research also suggests a strong positive correlation between 『trunk- based development,』 『no long-lived dev branches,』 and good technical outcomes. The underlying idea in both of those ideas seems clear: branches are a drag on productivity. In many cases we think complex branch and merge strategies are a perceived safety crutch—an attempt to keep trunk stable. As we see throughout this book, there are other ways to achieve that outcome.

同樣的DORA研究也表明，"基於主幹的開發"、"沒有長期的開發分支"和良好的技術成果之間有很強的正相關關係。這兩個觀點的基本思路似乎都很清楚：分支拖累了生產力。在許多情況下，我們認為複雜的分支和合並策略是一種可感知的安全支柱——試圖保持主幹的穩定。正如我們在本書中所看到的，還有其他的方法來實現這一結果。

## Version Control at Google  谷歌的版本控制

At Google, the vast majority of our source is managed in a single repository (monorepo) shared among roughly 50,000 engineers. Almost all projects that are owned by Google live there, except large open source projects like Chromium and Android. This includes public-facing products like Search, Gmail, our advertising products, our Google Cloud Platform offerings, as well as the internal infrastructure necessary to support and develop all of those products.

在谷歌，我們的絕大多數原始碼都在一個由大約50,000名工程師共享的儲存庫（monorepo）中管理。除了像Chromium和Android這樣的大型開源專案，幾乎所有屬於谷歌的專案都在這裡。這包括面向公眾的產品，如搜尋、Gmail、我們的廣告產品、我們的谷歌雲平台產品，以及支援和開發所有這些產品所需的內部基礎設施。

We rely on an in-house-developed centralized VCS called Piper, built to run as a distributed microservice in our production environment. This has allowed us to use Google-standard storage, communication, and Compute as a Service technology to provide a globally available VCS storing more than 80 TB of content and metadata. The Piper monorepo is then simultaneously edited and committed to by many thousands of engineers every day. Between humans and semiautomated processes that make use of version control (or improve things checked into VCS), we」ll regularly handle 60,000 to 70,000 commits to the repository per work day. Binary artifacts are fairly common because the full repository isn」t transmitted and thus the normal costs of binary artifacts don」t really apply. Because of the focus on Google-scale from the earliest conception, operations in this VCS ecosystem are still cheap at human scale: it takes perhaps 15 seconds total to create a new client at trunk, add a file, and commit an (unreviewed) change to Piper. This low-latency interaction and well-understood/ well-designed scaling simplifies a lot of the developer experience.

我們依靠內部開發的集中式VCS，名為Piper，該VCS是為在我們的生產環境中作為分散式微服務執行而建立的。這使我們能夠使用谷歌標準的儲存、通訊和計算即服務技術，提供一個全球可用的VCS，儲存超過80TB的內容和元資料。然後，Piper 單版本庫每天由成千上萬的工程師同時進行編輯和送出。在人類和利用版本控制（或改進簽入VCS的內容）的人工流程和半自動化流程之間，我們每個工作日會定期處理60,000到70,000次送出到版本庫。二進位構件是相當常見的，因為並不需要完整地傳輸到版本庫，因此二進位構件的成本並不高。由於從最初的概念就專注於谷歌規模，這個VCS生態系統的操作在人群規模上仍然是低成本的：在主幹上建立一個新的用戶端，新增一個檔案，並向Piper送出一個（未經審查的）更改，總共可能需要15秒。這種低延遲的互動和良好的理解/設計的擴充套件簡化了很多開發者的體驗。

By virtue of Piper being an in-house product, we have the ability to customize it and enforce whatever source control policies we choose. For instance, we have a notion of granular ownership in the monorepo: at every level of the file hierarchy, we can find OWNERS files that list the usernames of engineers that are allowed to approve commits within that subtree of the repository (in addition to the OWNERS that are listed at higher levels in the tree). In an environment with many repositories, this might have been achieved by having separate repositories with filesystem permissions enforcement controlling commit access or via a Git 『commit hook』 (action triggered at commit time) to do a separate permissions check. By controlling the VCS, we can make the concept of ownership and approval more explicit and enforced by the VCS during an attempted commit operation. The model is also flexible: ownership is just a text file, not tied to a physical separation of repositories, so it is trivial to update as the result of a team transfer or organization restructuring.

由於Piper是一個內部產品，我們能夠定製它並實施我們選擇的任何原始碼控制策略。例如，我們在單版本庫中有一個細粒度所有權的概念：在檔案層次結構的每一級，我們都可以找到OWNERS檔案，其中列出了允許批准該版本庫的子樹中的送出的工程師的使用者名（除了在樹中更高層次列出的OWNERS）。在具有多個版本庫的環境中，這可能是透過單獨的版本庫和檔案系統許可權執行控制送出訪問，或者透過Git的 "送出鉤子"（送出時觸發的動作）進行單獨的許可權檢查來實現。透過控制VCS，我們可以使所有權和批准的概念更加明確，並在嘗試送出操作時由VCS強制執行。這個模型也很靈活：所有權只是一個文字檔案，並不與儲存庫的物理分離相聯系，所以在團隊轉移或組織結構調整的情況下，更新它是很容易的。

### One Version  一個版本

The incredible scaling powers of Piper alone wouldn」t allow the sort of collaboration that we rely upon. As we said earlier: version control is also about policy. In addition to our VCS, one key feature of Google」s version control policy is what we」ve come to refer to as 『One Version.』 This extends the 『Single Source of Truth』 concept we looked at earlier—ensuring that a developer knows which branch and repository is their source of truth—to something like 『For every dependency in our repository, there must be only one version of that dependency to choose.』[^9] For third-party packages, this means that there can be only a single version of that package checked into our repository, in the steady state.[^10] For internal packages, this means no forking without repackaging/renaming: it must be technologically safe to mix both the original and the fork into the same project with no special effort. This is a powerful feature for our ecosystem: there are very few packages with restrictions like 『If you include this package (A), you cannot include other package (B).』

單憑Piper令人難以置信的擴充套件能力，是無法實現我們所依賴的那種協作的。正如我們之前所說：版本控制也是關於策略的。除了我們的VCS之外，谷歌版本控制策略的一個關鍵特徵就是我們所說的 "一個版本"。這擴充套件了我們前面提到的 "單訊息源 "的概念--確保開發者知道哪個分支和版本庫是他們的訊息源--到類似於 "對於我們版本庫中的每個依賴，必須只有一個版本的依賴可以選擇。 "對於第三方軟體套件，這意味著在穩定狀態下，該軟體套件只能有一個版本被檢入我們的倉庫。對於內部軟體套件，這意味著沒有重新打包/重新命名的分支：在技術上必須是安全的，無需特別努力就可以將原始和分支混合到同一個專案中。這對我們的生態系統來說是一個強大的功能：很少有套件有類似 "如果你包括這個軟體套件（A），你就不能包括其他軟體套件（B）"的限制。

This notion of having a single copy on a single branch in a single repository as our Source of Truth is intuitive but also has some subtle depth in application. Let」s investigate a scenario in which we have a monorepo (and thus arguably have fulfilled the letter of the law on Single Source of Truth), but have allowed forks of our libraries to propagate on trunk.

將單個副本放在單個版本庫中的單個分支上作為訊息源的概念是直觀的，但在應用中也有一些微妙的深度。讓我們研究一下這樣的場景：我們有一個單版本庫（因此可以說已經履行了關於單訊息源的法律條文），但允許我們的函式庫的分支在主幹上傳播。

> [^9]: For example, during an upgrade operation, there might be two versions checked in, but if a developer is adding a new dependency on an existing package, there should be no choice in which version to depend upon.
>
> 9  例如，在升級操作期間，可能簽入了兩個版本，但如果開發人員正在現有軟體套件上新增新的依賴，則應該沒有選擇依賴哪個版本。
>
> [^10]: That said, we fail at this in many cases because external packages sometimes have pinned copies of their own dependencies bundled in their source release. You can read more on how all of this goes wrong in Chapter 21.
>
> 10 也就是說，我們在很多情況下都會失敗，因為外部軟體套件有時會在它們的源版本中捆綁有它們自己的依賴性的釘子副本。你可以在第21章中閱讀更多關於這一切是如何出錯的。

### Scenario: Multiple Available Versions  場景：多個可用版本

Imagine the following scenario: some team discovers a bug in common infrastructure code (in our case, Abseil or Guava or the like). Rather than fix it in place, the team decides to fork that infrastructure and tweak it to work around the bug—without renaming the library or the symbols. It informs other teams near them, 『Hey, we have an improved version of Abseil checked in over here: check it out.』 A few other teams build libraries that themselves rely on this new fork.

想象一下以下情況：一些團隊發現了公共基礎元件程式碼中的一個bug（在我們的例子中，是Abseil或Guava之類的）。該團隊決定不在原地修復它，而是分支該基礎元件，並對其進行調整，以解決該錯誤——而不重新命名函式庫或符號。它通知他們附近的其他團隊："嘿，我們這裡有一個改進的Abseil版本：請檢視。" 其他一些團隊建立的函式庫也依賴於這個新的分支。

As we」ll see in Chapter 21, we」re now in a dangerous situation. If any project in the codebase comes to depend on both the original and the forked versions of Abseil simultaneously, in the best case, the build fails. In the worst case, we」ll be subjected to difficult-to-understand runtime bugs stemming from linking in two mismatched versions of the same library. The 『fork』 has effectively added a coloring/partitioning property to the codebase: the transitive dependency set for any given target must include exactly one copy of this library. Any link added from the 『original flavor』 partition of the codebase to the 『new fork』 partition will likely break things. This means that in the end that something as simple as 『adding a new dependency』 becomes an operation that might require running all tests for the entire codebase, to ensure that we haven」t violated one of these partitioning requirements. That」s expensive, unfortunate, and doesn」t scale well.

正如我們將在第21章中看到的，我們現在處於危險的境地。如果程式碼庫中的任何專案同時依賴Abseil的原始版本和分支版本，在最好的情況下，建立將失敗。在最壞的情況下，我們將受到難以理解的執行時錯誤的影響，這些錯誤源於同一個函式庫的兩個不對應的版本的連結。『fork』有效地為程式碼庫新增了一個著色/分割槽屬性：任何給定目標的可傳遞依賴項集必須只包含該函式庫的一個副本。從『原始味道』的程式碼庫新增到『新分支』分割槽的任何連結都可能會破壞事物。這意味著到最後，像 "新增一個新的依賴"這樣簡單的操作，可能需要執行整個程式碼庫的所有測試，以確保我們沒有違反這些分割槽的要求。這很昂貴，很不幸，而且不能很好地擴充套件。

In some cases, we might be able to hack things together in a way to allow a resulting executable to function correctly. Java, for instance, has a relatively standard practice called [*shading*](https://oreil.ly/RuWX3), which tweaks the names of the internal dependencies of a library to hide those dependencies from the rest of the application. When dealing with functions, this is technically sound, even if it is theoretically a bit of a hack. When dealing with types that can be passed from one package to another, shading solutions work neither in theory nor in practice. As far as we know, any technological trickery that allows multiple isolated versions of a library to function in the same binary share this limitation: that approach will work for functions, but there is no good (efficient) solution to shading types—multiple versions for any library that provides a vocabulary type (or any higher-level construct) will fail. Shading and related approaches are patching over the underlying issue: multiple versions of the same dependency are needed. (We」ll discuss how to minimize that in general in Chapter 21.)

在某些情況下，我們也許可以透過駭客技術將一些東西拼湊在一起，使產生的可執行檔案能夠正常執行。例如，Java有一個相對標準的做法，叫做[*shading*](https://oreil.ly/RuWX3)，它調整了函式庫的內部依賴的名稱，以便從應用程式的其他部分隱藏這些依賴關係。當處理函式時，這在技術上是合理的，即使它在理論上有點像駭客。當處理可以從一個套件傳遞到另一個套件的型別時，著色解決方案在理論上和實踐中都不起作用。據我們所知，任何允許一個函式庫的多個孤立版本在同一個二進位中運作的技術伎倆都有這個限制：這種方法對函式來說是可行的，但對於著色型別來說，沒有好的（有效的）解決方案——任何提供詞彙型別（或任何更高階別的構造）的函式庫的多個版本都會失敗。著色和相關的方法是對基本問題的修補：同一依賴的多個版本是需要的。(我們將在第21章中討論如何在一般情況下儘量減少這種情況)。

Any policy system that allows for multiple versions in the same codebase is allowing for the possibility of these costly incompatibilities. It」s possible that you」ll get away with it for a while (we certainly have a number of small violations of this policy), but in general, any multiple-version situation has a very real possibility of leading to big problems.

任何允許在同一程式碼庫中使用多個版本的策略系統都可能會出現這些代價高昂的不相容。你有可能暫時逃過一劫（我們當然有一些小的違反這一策略的行為），但一般來說，任何多版本的情況都有導致大問題的非常現實的可能性。

### The 『One-Version』 Rule  『一個版本』規則

With that example in mind, on top of the Single Source of Truth model, we can hopefully understand the depth of this seemingly simple rule for source control and branch management:

考慮到這個例子，在單訊息源模型的基礎上，我們希望能夠充分理解這一看似簡單的原始碼控制和分支管理規則的深度：

    Developers must never have a choice of 『What version of this component should I depend upon?』
    決不能讓開發人員選擇"我應該依賴這個元件的哪個版本？"

Colloquially, this becomes something like a 『One-Version Rule.』 In practice, 『One- Version』 is not hard and fast,[^11] but phrasing this around limiting the versions that can be *chosen* when adding a new dependency conveys a very powerful understanding.

俗話說，這就變成了類似於 "一個版本規則"的東西。在實踐中，"一個版本"並不是硬性規定，但在新增新依賴項時限制可以選擇的版本這一措辭傳達了一種非常有力的理解。

For an individual developer, lack of choice can seem like an arbitrary impediment. Yet we see again and again that for an organization, it」s a critical component in efficient scaling. Consistency has a profound importance at all levels in an organization. From one perspective, this is a direct side effect of discussions about consistency and ensuring the ability to leverage consistent 『choke points.』

對於個人開發者來說，缺乏選擇似乎是一個大障礙。然而，我們一再看到，對於一個組織來說，它是高效擴充套件的一個關鍵組成部分。一致性在一個組織的各個層面都有深遠的意義。從一個角度來看，這是討論一致性和確保利用一致 "瓶頸"的能力的直接副作用。

> [^11]: For instance, if there are external/third-party libraries that are periodically updated, it might be infeasible to update that library and update all use of it in a single atomic change. As such, it is often necessary to add a new version of that library, prevent new users from adding dependencies on the old one, and incrementally switch usage from old to new.
>
> 11 例如，如果有定期更新的外部/第三方庫，更新該函式庫並在一次原子變化中更新對它的所有使用可能是不可行的。因此，通常有必要新增該函式庫的新版本，防止新使用者新增對舊版本的依賴，並逐步將使用從舊版本切換到新版本。

### (Nearly) No Long-Lived Branches  (幾乎)沒有長期存在的分支

There are several deeper ideas and policies implicit in our One-Version Rule; foremost among them: development branches should be minimal, or at best be very short lived. This follows from a lot of published work over the past 20 years, from Agile processes to DORA research results on trunk-based development and even Phoenix Project[^12] lessons on 『reducing work-in-progress.』 When we include the idea of pending work as akin to a dev branch, this further reinforces that work should be done in small increments against trunk, committed regularly.

在我們的 "一個版本規則"中隱含著幾個更深層次的想法和策略；其中最重要的是：開發分支應該是最小的，或者最多只能是很短的時間。這來自於過去20年裡發表的大量工作，從敏捷過程到基於主幹的開發的DORA研究成果，甚至鳳凰計劃關於 "減少進行中的工作"的教訓。當我們把待完成的工作看作是類似於開發分支的想法時，這就進一步強化了工作應該針對主幹，定期送出，以小的增量完成。

As a counterexample: in a development community that depends heavily on long- lived development branches, it isn」t difficult to imagine opportunity for choice creeping back in.

作為一個反例：在一個嚴重依賴長期存在的開發分支的開發社群，不難想象選擇的場景又悄然而至。

Imagine this scenario: some infrastructure team is working on a new Widget, better than the old one. Excitement grows. Other newly started projects ask, 『Can we depend on your new Widget?』 Obviously, this can be handled if you」ve invested in codebase visibility policies, but the deep problem happens when the new Widget is 『allowed』 but only exists in a parallel branch. Remember: new development must not have a choice when adding a dependency. That new Widget should be committed to trunk, disabled from the runtime until it」s ready, and hidden from other developers by visibility if possible—or the two Widget options should be designed such that they can coexist, linked into the same programs.

想象一下這樣的場景：一些基礎元件團隊正在開發一個新的Widget，比老的更好。興奮之情油然而生。其他新開始的專案問："我們可以依賴你的新Widget嗎？" 顯然，如果你在程式碼庫的可見性策略上進行了投資，這種情況是可以處理的，但當新的Widget被 "允許"，深層次的問題就會發生但只存在於並行分支中。記住：新的開發在新增依賴關係時不能有選擇。那個新的Widget應該被送出到主幹，在它準備好之前被禁止在執行時使用，並且如果可能的話，透過可見性來隱藏其他開發者，或者兩個Widget選項應該被設計成它們可以共存，被連結到同一個程式中。

Interestingly, there is already evidence of this being important in the industry. In Accelerate and the most recent State of DevOps reports, DORA points out that there is a predictive relationship between trunk-based development and high-performing software organizations. Google is not the only organization to have discovered this— nor did we necessarily have expected outcomes in mind when these policies evolved —--—it just seemed like nothing else worked. DORA」s result certainly matches our experience.

有趣的是，已經有證據表明這在行業中是很重要的。在《加速》和最近的《DevOps狀況》報告中，DORA指出，基於主幹的開發和高績效的軟體組織之間存在著可預測關係。谷歌並不是唯一發現這一點的組織--當這些策略演變時，我們也不一定有預期的結果——只是看起來沒有別的辦法了。DORA的結果當然與我們的經驗相符。

Our policies and tools for large-scale changes (LSCs; see [Chapter 22](#_bookmark1935)) put additional weight on the importance of trunk-based development: broad/shallow changes that are applied across the codebase are already a massive (often tedious) undertaking when modifying everything checked in to the trunk branch. Having an unbounded number of additional dev branches that might need to be refactored at the same time would be an awfully large tax on executing those types of changes, finding an ever- expanding set of hidden branches. In a DVCS model, it might not even be possible to identify all of those branches.

我們的大規模變更（LSCs；見第22章）的策略和工具給基於主幹的開發的重要性增加了砝碼：當修改所有簽入主幹分支的內容時，適用於整個程式碼庫的廣泛/淺層變更已經是一項巨大的（通常是乏味的）工作。如果在同一時間有數量不限的額外開發分支需要被重構，那麼對於執行這些型別的修改來說，將是一個非常大的負擔，因為要找到一組不斷擴大的隱藏分支。在DVCS模型中，甚至可能無法識別所有這些分支。

Of course, our experience is not universal. You might find yourself in unusual situations that require longer-lived dev branches in parallel to (and regularly merged with) trunk.

當然，我們的經驗並不是萬能的。你可能會發現自己處於不尋常的情況下，需要更長的開發分支與主幹並行（並定期合併）。

Those scenarios should be rare, and should be understood to be expensive. Across the roughly 1,000 teams that work in the Google monorepo, there are only a couple that have such a dev branch.[^13] Usually these exist for a very specific (and very unusual) reason. Most of those reasons boil down to some variation of 『We have an unusual requirement for compatibility over time.』 Oftentimes this is a matter of ensuring compatibility for data at rest across versions: readers and writers of some file format need to agree on that format over time even if the reader or writer implementations are modified. Other times, long-lived dev branches might come from promising API compatibility over time—when One Version isn」t enough and we need to promise that an older version of a microservice client still works with a newer server (or vice versa). That can be a very challenging requirement, something that you should not promise lightly for an actively evolving API, and something you should treat carefully to ensure that period of time doesn」t accidentally begin to grow. Dependency across time in any form is far more costly and complicated than code that is time invariant. Internally, Google production services make relatively few promises of that form.[^14] We also benefit greatly from a cap on potential version skew imposed by our 『build horizon』: every job in production needs to be rebuilt and redeployed every six months, maximum. (Usually it is far more frequent than that.)

這些場景應該是罕見的，並且應該理解為代價高昂。在谷歌單版本庫的大約1000個團隊中，只有少數團隊有這樣一個開發分支。這些場景的存在通常有一個非常具體（非常不尋常）的原因。大多數原因歸結為『隨著時間的推移，我們對相容性有著苛刻的要求。』通常，這是一個確保跨版本的靜態資料的相容性的問題：某些檔案格式的讀寫器需要隨著時間的推移對該格式達成一致意見，即使讀寫器實現被修改。其他時候，長期的開發分支可能來自於對API相容性的承諾--當一個版本還不夠時，我們需要承諾舊版本的微服務用戶端仍能與新版本的伺服器相容（反之亦然）。這可能是一個非常具有挑戰性的要求，對於一個積極發展的API，你不應該輕易承諾，而且你應該謹慎對待，以確保這段時間不會意外地開始增長。任何形式的跨時間的依賴都比時間不變的程式碼要昂貴和複雜得多。在內部，谷歌生產服務相對來說很少做出這種形式的承諾。我們也從我們的 "建立範圍 "所施加的潛在版本偏差上限中獲益匪淺：生產中的每項工作最多每六個月就需要重建和重新部署。(通常要比這頻繁得多）。

We」re sure there are other situations that might necessitate long-lived dev branches. Just make sure to keep them rare. If you adopt other tools and practices discussed in this book, many will tend to exert pressure against long-lived dev branches. Automation and tooling that works great at trunk and fails (or takes more effort) for a dev branch can help encourage developers to stay current.

我們確信還有其他情況可能需要長期的開發分支。只需確保它們很少。如果你採用了本書所討論的其他工具和實踐，很多人都會傾向於對長期的開發分支施加壓力。自動化和工具在主幹分支上執行良好，而在開發分支上則失敗（或花費更多精力），這有助於鼓勵開發人員保持更新。

> [^12]: Kevin Behr, Gene Kim, and George Spafford, The Phoenix Project (Portland: IT Revolution Press, 2018).
> 12  Kevin Behr、Gene Kim和George Spafford，《鳳凰城專案》（波特蘭：IT革命出版社，2018年）。
>
> [^13]: It」s difficult to get a precise count, but the number of such teams is almost certainly fewer than 10./
> 13  很難準確統計，但這樣的隊伍幾乎肯定少於10支。
>
> [^14]: Cloud interfaces are a different story.
> 14  雲介面是另一回事。

#### What About Release Branches?  發布分支呢？

Many Google teams use release branches, with limited cherry picks. If you」re going to put out a monthly release and continue working toward the next release, it」s perfectly reasonable to make a release branch. Similarly, if you」re going to ship devices to customers, it」s valuable to know exactly what version is out 『in the field.』 Use caution and reason, keep cherry picks to a minimum, and don」t plan to remerge with trunk. Our various teams have all sorts of policies about release branches given that relatively few teams have arrived at the sort of rapid release cadence promised by CD (see Chapter 24) that obviates the need or desire for a release branch. Generally speaking,release branches don」t cause any widespread cost in our experience. Or, at least, no noticeable cost above and beyond the additional inherent cost to the VCS.

許多谷歌團隊使用發布分支，但選擇的版本有限。如果你打算每月發布一個版本，並繼續為下一個版本工作，那麼建立一個發布分支是完全合理的。同樣，如果你打算將裝置交付給客戶，準確地知道什麼版本『在當前』是很有價值的。謹慎和理智，儘量減少偷樑換柱的行為，並且不要計劃與主幹分支重新合併。鑑於很少有團隊達到CD承諾的快速發布節奏，我們的各個團隊對發布分支有各種各樣的策略（見第24章）這樣就不需要或不需要發布分支。一般來說，根據我們的經驗，發布分支不會導致任何廣泛的成本。或者說，至少在VCS的額外固有成本之外，沒有明顯的成本。

## Monorepos    單版本庫（單庫）

In 2016, we published a (highly cited, much discussed) paper on Google」s monorepo approach.[^15] The monorepo approach has some inherent benefits, and chief among them is that adhering to One Version is trivial: it」s usually more difficult to violate One Version than it would be to do the right thing. There」s no process of deciding which versions of anything are official, or discovering which repositories are important. Building tools to understand the state of the build (see Chapter 23) doesn」t also require discovering where important repositories exist. Consistency helps scale up the impact of introducing new tools and optimizations. By and large, engineers can see what everyone else is doing and use that to inform their own choices in code and system design. These are all very good things.

2016年，我們發表了一篇關於Google的單版本庫（方法的論文（引用率很高，討論很多）。monorepo方法有一些固有的好處，其中最主要的是遵守一個版本是微不足道的：通常違反一個版本比做正確的事情更難。沒有過程來決定任何東西的哪個版本是官方的，也沒有發現哪個版本庫是重要的。建立工具來了解構建的狀態（見第23章）也不需要發現哪裡有重要的軟體庫。一致性有助於擴大引入新工具和最佳化的影響。總的來說，工程師們可以看到其他人在做什麼，並利用這些來告知他們自己在程式碼和系統設計中的選擇。這些都是非常好的事情。

Given all of that and our belief in the merits of the One-Version Rule, it is reasonable to ask whether a monorepo is the One True Way. By comparison, the open source community seems to work just fine with a 『manyrepo』 approach built on a seemingly infinite number of noncoordinating and nonsynchronized project repositories.

考慮到所有這些，以及我們對 "單一版本規則 "優點的信念，我們有理由問，單版本庫是否是唯一正確的方法。相比之下，開源社群似乎可以用 "多版本 "的方法來工作，而這種方法是建立在看似無限多的不協調和不同步的專案庫之上的。

In short: no, we don」t think the monorepo approach as we」ve described it is the perfect answer for everyone. Continuing the parallel between filesystem format and VCS, it」s easy to imagine deciding between using 10 drives to provide one very large logical filesystem or 10 smaller filesystems accessed separately. In a filesystem world, there are pros and cons to both. Technical issues when evaluating filesystem choice would range from outage resilience, size constraints, performance characteristics, and so on. Usability issues would likely focus more on the ability to reference files across filesystem boundaries, add symlinks, and synchronize files.

簡而言之：不，我們不認為我們所描述的單版本庫方法對每個人都是完美答案。持續檔案系統格式和VCS之間的並行，很容易想象在使用10個驅動器提供一個非常大的邏輯檔案系統還是10個單獨訪問的小檔案系統之間做出決定。在檔案系統的世界裡，兩者都有優點和缺點。在評估檔案系統的選擇時，技術上的問題包括中斷恢復能力、大小限制、效能特點等等。可用性問題可能會更多地集中在跨檔案系統邊界引用檔案、新增符號連結和同步檔案的能力上。

A very similar set of issues governs whether to prefer a monorepo or a collection of finer-grained repositories. The specific decisions of how to store your source code (or store your files, for that matter) are easily debatable, and in some cases, the particulars of your organization and your workflow are going to matter more than others. These are decisions you」ll need to make yourself.

一組非常類似的問題決定了是選擇單版本庫（還是選擇更細粒度的版本庫的集合。如何儲存你的原始碼（或儲存你的檔案）的具體決定是很容易爭論的，在某些情況下，你的組織和你的工作流程的特殊性會比其他的更重要。這些都是你需要自己做出的決定。

What is important is not whether we focus on monorepo; it」s to adhere to the One- Version principle to the greatest extent possible: developers must not have a *choice* when adding a dependency onto some library that is already in use in the organization. Choice violations of the One-Version Rule lead to merge strategy discussions, diamond dependencies, lost work, and wasted effort.

重要的不是我們是否關注單版本庫；而是最大限度地堅持一個版本的原則：開發人員在向組織中已經使用的某個函式庫新增依賴時，不能有*選擇*。違反一個版本原則的選擇會導致合併策略的討論、鑽石依賴、工作損失和工作消耗。

Software engineering tools including both VCS and build systems are increasingly providing mechanisms to smartly blend between fine-grained repositories and monorepos to provide an experience akin to the monorepo—an agreed-upon ordering of commits and understanding of the dependency graph. Git submodules, Bazel with external dependencies, and CMake subprojects all allow modern developers to synthesize something weakly approximating monorepo behavior without the costs and downsides of a monorepo.[^16] For instance, fine-grained repositories are easier to deal with in terms of scale (Git often has performance issues after a few million commits and tends to be slow to clone when repositories include large binary artifacts) and storage (VCS metadata can add up, especially if you have binary artifacts in your version control system). Fine-grained repositories in a federated/virtual-monorepo (VMR)–style repository can make it easier to isolate experimental or top-secret projects while still holding to One Version and allowing access to common utilities.

包括VCS和建立系統在內的軟體工程工具越來越多地提供了在細粒度版本庫和單版本庫之間巧妙融合的機制，以提供類似於單版本庫的體驗--一種約定的送出順序和對依賴關係圖的理解。Git子模組、帶有外部依賴關係的Bazel和CMake子專案都允許現代開發者合成一些弱的近似於單版本庫的行為，而沒有單版本庫的成本和弊端。例如，細粒度的版本庫在規模上更容易處理（Git在幾百萬次送出後經常出現效能問題，而且當倉庫包括大型二進位構件時，克隆速度往往很慢）和儲存（VCS元資料會增加，特別是如果你的版本控制系統中有二進位構件）。聯合/虛擬單版本庫（VMR）風格的細粒度版本庫可以更容易地隔離實驗性或最高機密的專案，同時同時仍保留一個版本並允許訪問通用工具。

To put it another way: if every project in your organization has the same secrecy, legal, privacy, and security requirements,[^17] a true monorepo is a fine way to go. Otherwise, *aim* for the functionality of a monorepo, but allow yourself the flexibility of implementing that experience in a different fashion. If you can manage with disjoint repositories and adhere to One Version or your workload is all disconnected enough to allow truly separate repositories, great. Otherwise, synthesizing something like a VMR in some fashion may represent the best of both worlds.

換言之：如果你組織中的每個專案都有相同的保密、法律、隱私和安全要求，真正的單版本庫是一個不錯的選擇。否則，以單版本庫的功能為目標，但允許自己以不同的方式靈活實施該體驗。如果你可以用不相干的軟體庫來管理，並且堅持一個版本，或者你的工作量都是不相干的，足以允許真正的獨立軟體庫，那就太好了。否則，以某種方式合成類似於VMR的東西可能代表了兩個世界的最佳狀態。

After all, your choice of filesystem format really doesn」t matter as much as what you write to it.

畢竟，你對檔案系統格式的選擇與你向其寫入的內容相比，真的並不重要。

> [^15]: Rachel Potvin and Josh Levenberg, 『Why Google stores billions of lines of code in a single repository,』 Communications of the ACM, 59 No. 7 (2016): 78-87.
>
> 15 Rachel Potvin和Josh Levenberg，"為什麼谷歌將數十億行程式碼儲存在一個庫中，"《ACM通訊》，59 No.7（2016）：78-87。
>
> [^16]: We don」t think we」ve seen anything do this particularly smoothly, but the interrepository dependencies/virtual monorepo idea is clearly in the air.
>
> 16 我們認為我們還沒有看到任何系統能特別順利地做到這一點，但版本庫間的依賴關係/虛擬單庫的想法顯然是在空想中。
>
> [^17]: Or you have the willingness and capability to customize your VCS—and maintain that customization for the lifetime of your codebase/organization. Then again, maybe don」t plan on that as an option; that is a lot of overhead.
>
> 17 或者你有意願和能力來定製你的VCS--並且在你的程式碼庫/組織的生命週期內保持這種定製。然後，也許不要把它作為一個選項，那是一個很大的開銷。

## Future of Version Control  版本控制的未來

Google isn」t the only organization to publicly discuss the benefits of a monorepo approach. Microsoft, Facebook, Netflix, and Uber have also publicly mentioned their reliance on the approach. DORA has published about it extensively. It」s vaguely possible that all of these successful, long-lived companies are misguided, or at least that their situations are sufficiently different as to be inapplicable to the average smaller organization. Although it」s possible, we think it is unlikely.

谷歌並不是唯一一個公開討論單版本庫方法的好處的組織。微軟、Facebook、Netflix和Uber也公開提到他們對這種方法的依賴。DORA已經廣泛地發表了關於它的文章。很可能所有這些成功的、長期存在的公司都被誤導了，或者至少他們的情況差異很大，不適用於一般較小的組織。雖然這是可能的，但我們認為不太可能。

Most arguments against monorepos focus on the technical limitations of having a single large repository. If cloning a repository from upstream is quick and cheap, developers are more likely to keep changes small and isolated (and to avoid making mistakes with committing to the wrong work-in-progress branch). If cloning a repository (or doing some other common VCS operation) takes hours of wasted developer time, you can easily see why an organization would shy away from reliance on such a large repository/operation. We luckily avoided this pitfall by focusing on providing a VCS that scales massively.

大多數反對單版本庫的論點都集中在擁有一個大型版本庫的技術限制上。如果從上游克隆一個版本庫又快又便宜，開發者就更有可能保持小規模和隔離的更改（避免送出到錯誤的工作分支）。如果克隆一個版本庫（或做一些其他常見的VCS操作）需要浪費開發人員幾個小時的時間，你很容易理解為什麼一個組織會避開對這種大型版本庫/操作的依賴。我們很幸運地避免了這個陷阱，因為我們專注於提供一個可以大規模擴充套件的VCS。

Looking at the past few years of major improvements to Git, there」s clearly a lot of work being done to support larger repositories: shallow clones, sparse branches, better optimization, and more. We expect this to continue and the importance of 『but we need to keep the repository small』 to diminish.

回顧過去幾年對Git的重大改進，顯然有很多工作是為了支援更大的倉庫：淺複製，稀疏分支，更好的最佳化，等等。我們希望這種情況能繼續下去，而 "但我們需要保持倉庫的小型化"的重要性則會降低。

The other major argument against monorepos is that it doesn」t match how development happens in the Open Source Software (OSS) world. Although true, many of the practices in the OSS world come (rightly) from prioritizing freedom, lack of coordination, and lack of computing resources. Separate projects in the OSS world are effectively separate organizations that happen to be able to see one another」s code. Within the boundaries of an organization, we can make more assumptions: we can assume the availability of compute resources, we can assume coordination, and we can assume that there is some amount of centralized authority.

反對單版本庫的另一個主要論點是，它不符合開源軟體（OSS）世界中的開發方式。雖然這是事實，但開放原始碼軟體世界中的許多做法（正確地）來自於對自由的優先考慮，缺乏協調，以及缺乏計算資源。在開放原始碼軟體世界中，獨立的專案實際上是獨立的組織，碰巧可以看到彼此的程式碼。在一個組織的邊界內，我們可以做出更多的假設：我們可以假設計算資源的可用性，我們可以假設協調，我們可以假設有一定程度的集中許可權。

A less common but perhaps more legitimate concern with the monorepo approach is that as your organization scales up, it is less and less likely that every piece of code is subject to exactly the same legal, compliance, regulatory, secrecy, and privacy requirements. One native advantage of a manyrepo approach is that separate repositories are obviously capable of having different sets of authorized developers, visibility, permissions, and so on. Stitching that feature into a monorepo can be done but implies some ongoing carrying costs in terms of customization and maintenance.

對於單版本庫的方法，一個不太常見但也許更合理的擔憂是，隨著你的組織規模的擴大，越來越不可能每段程式碼都受到完全相同的法律、合規、監管、保密和隱私要求的約束。多版本函式庫方法的一個原生優勢是，獨立的版本庫顯然能夠擁有不同的授權開發者、可見性、許可權等集合。整合這個功能到一個單庫中是可以做到的，但意味著在定製和維護方面有一些持續的承載成本。

At the same time, the industry seems to be inventing lightweight interrepository linkage over and over again. Sometimes, this is in the VCS (Git submodules) or the build system. So long as a collection of repositories have a consistent understanding of 『what is trunk,』 『which change happened first,』 and mechanisms to describe dependencies, we can easily imagine stitching together a disparate collection of physical repositories into one larger VMR. Even though Piper has done very well for us, investing in a highly scaling VMR and tools to manage it and relying on off-the-shelf customization for per-repository policy requirements could have been a better investment.

與此同時，業界似乎在一次又一次地發明輕量級的庫間連結。有時，這是在VCS（Git子模組）或建立系統中。只要版本庫的集合對 "什麼是主幹"、"哪個變化先發生 "有一致的理解，並有描述依賴關係的機制，我們就可以很容易地想象把不同的物理版本庫的集合縫合到一個更大的VMR中。儘管Piper為我們做得很好，但投資於一個高度擴充套件的VMR和工具來管理它，並依靠現成的定製來滿足每個版本庫的策略要求，可能是一個更好的投資。

As soon as someone builds a sufficiently large nugget of compatible and interdependent projects in the OSS community and publishes a VMR view of those packages, we suspect that OSS developer practices will begin to change. We see glimpses of this in the tools that *could* synthesize a virtual monorepo as well as in the work done by (for instance) large Linux distributions discovering and publishing mutually compatible revisions of thousands of packages. With unit tests, CI, and automatic version bumping for new submissions to one of those revisions, enabling a package owner to update trunk for their package (in nonbreaking fashion, of course), we think that model will catch on in the open source world. It is just a matter of efficiency, after all: a (virtual) monorepo approach with a One-Version Rule cuts down the complexity of software development by a whole (difficult) dimension: time.

一旦有人在開放原始碼軟體社群建立了足夠大的相容和相互依賴的專案，併發布了這些軟體套件的VMR檢視，我們懷疑開放原始碼軟體開發者的做法將開始改變。我們在*能*合成虛擬單一版本庫的工具中，以及在（例如）大型Linux發行版發現和發布數千個軟體套件的相互相容的修訂版所做的工作中看到了這一跡象。有了單元測試、CI，以及對其中一個修訂版的新送出的自動版本升級，使軟體套件所有者能夠為他們的軟體套件更新主幹（當然是以不破壞的方式），我們認為這種模式將在開源世界中流行起來。畢竟，這只是一個效率問題：一個（虛擬的）單一版本的方法與一個版本的規則，將軟體開發的複雜性減少了一整個（困難的）層面：時間。

We expect version control and dependency management to evolve in this direction in the next 10 to 20 years: VCSs will focus on *allowing* larger repositories with better performance scaling, but also removing the need for larger repositories by providing better mechanisms to stitch them together across project and organizational boundaries. Someone, perhaps the existing package management groups or Linux distributors, will catalyze a de facto standard virtual monorepo. Depending on the utilities in that monorepo will provide easy access to a compatible set of dependencies as one unit. We」ll more generally recognize that version numbers are timestamps, and that allowing version skew adds a dimensionality complexity (time) that costs a lot—and that we can learn to avoid. It starts with something logically like a monorepo.

我們預計在未來10到20年內，版本控制和依賴管理將朝著這個方向發展。VCS將專注於允許*大型版本庫*，並有更好的效能擴充套件，但也透過提供更好的機制來消除對大版本函式庫的需求，使它們跨越專案和組織的界限。其中一個，也許是現有的軟體套件管理小組或Linux發行商，將促成一個事實上的標準虛擬單一版本庫。依靠單一版本庫中的實用程式，可以方便地訪問作為一個單元的相容的依賴關係。我們將更普遍地認識到，版本號是時間戳，允許版本偏差增加了一個維度的複雜性（時間），這需要花費很多，而且我們可以學習如何避免。它從邏輯上類似於單一版本庫的東西開始。

## Conclusion  總結

Version control systems are a natural extension of the collaboration challenges and opportunities provided by technology, especially shared compute resources and computer networks. They have historically evolved in lockstep with the norms of software engineering as we understand them at the time.

版本控制系統是技術帶來的協作挑戰和機遇的自然延伸，尤其是共享計算資源和計算機網絡。它們在歷史上與我們當時理解的軟體工程規範同步發展。

Early systems provided simplistic file-granularity locking. As typical software engineering projects and teams grew larger, the scaling problems with that approach became apparent, and our understanding of version control changed to match those challenges. Then, as development increasingly moved toward an OSS model with distributed contributors, VCSs became more decentralized. We expect a shift in VCS technology that assumes constant network availability, focusing more on storage and build in the cloud to avoid transmitting unnecessary files and artifacts. This is increasingly critical for large, long-lived software engineering projects, even if it means a change in approach compared to simple single-dev/single-machine programming projects. This shift to cloud will make concrete what has emerged with DVCS approaches: even if we allow distributed development, something must still be centrally recognized as the Source of Truth.

早期的系統提供了簡單的檔案細粒度鎖功能。隨著典型的軟體工程專案和團隊規模的擴大，這種方式的擴充套件問題變得很明顯，我們對版本控制的理解也隨著這些挑戰而改變。然後，隨著開發越來越多地轉向具有分散式貢獻者的開放原始碼軟體模型，VCS變得更加分散。我們期待著VCS技術的轉變，即假設網路的持續可用性，更加關注雲儲存和雲建立，以避免傳輸不必要的檔案和工件。這對於大型、長週期的軟體工程專案來說越來越關鍵，即使這意味著與簡單的單裝置/單機器程式設計專案相比，方法上的改變。這種向雲端運算的轉變將使DVCS方法中出現的內容具體化：即使我們允許分散式開發，也必須集中認識到某些東西是訊息源。

The current DVCS decentralization is a sensible reaction of the technology to the needs of the industry (especially the open source community). However, DVCS configuration needs to be tightly controlled and coupled with branch management policies that make sense for your organization. It also can often introduce unexpected scaling problems: perfect fidelity offline operation requires a lot more local data. Failure to rein in the potential complexity of a branching free-for-all can lead to a potentially unbounded amount of overhead between developers and deployment of that code. However, complex technology doesn」t need to be used in a complex fashion: as we see in monorepo and trunk-based development models, keeping branch policies simple generally leads to better engineering outcomes.

目前DVCS的去中心化是該技術對行業（尤其是開源社群）需求的合理反應。然而，DVCS的設定需要嚴格控制，並與對你的組織有意義的分支管理策略結合起來。它還常常會引入意想不到的擴充套件問題：完美模擬的離線操作需要更多的本地資料。如果不控制分支自由生成的潛在複雜性，就會導致開發人員和該程式碼的部署之間可能會出現無限開銷。然而，複雜的技術並不需要以複雜的方式來使用：正如我們在單一版本庫和基於主幹的開發模式中看到的那樣，保持分支策略的簡單通常會帶來更好的工程結果。

Choice leads to costs here. We highly endorse the One-Version Rule presented here: developers within an organization must not have a choice where to commit, or which version of an existing component to depend upon. There are few policies we」re aware of that can have such an impact on the organization: although it might be annoying for individual developers, in the aggregate, the end result is far better.

選擇帶來了成本。我們高度贊同這裡提出的 "單一版本規則"：組織內的開發者不能選擇送出到哪裡，或者選擇依賴現有元件的哪個版本。據我們所知，很少有策略能對組織產生如此大的影響：儘管這對個別開發者來說可能很煩人，但從總體上看，最終結果要好得多。

## TL;DRs  內容提要

- Use version control for any software development project larger than 『toy project with only one developer that will never be updated.』
- There」s an inherent scaling problem when there are choices in 『which version of this should I depend upon?』
- One-Version Rules are surprisingly important for organizational efficiency. Removing choices in where to commit or what to depend upon can result in significant simplification.
- In some languages, you might be able to spend some effort to dodge this with technical approaches like shading, separate compilation, linker hiding, and so on. The work to get those approaches working is entirely lost labor—your software engineers aren」t producing anything, they」re just working around technical debts.
- Previous research (DORA/State of DevOps/Accelerate) has shown that trunk- based development is a predictive factor in high-performing development organizations. Long-lived dev branches are not a good default plan.
- Use whatever version control system makes sense for you. If your organization wants to prioritize separate repositories for separate projects, it」s still probably wise for interrepository dependencies to be unpinned/『at head』/『trunk based.』 There are an increasing number of VCS and build system facilities that allow you to have both small, fine-grained repositories as well as a consistent 『virtual』 head/trunk notion for the whole organization.

- 對任何大於『只有一名開發人員且永遠不會更新的玩具專案』的軟體開發專案都要使用版本控制。
- 當存在 "我應該依賴哪個版本 "的選擇時，就會存在內在的擴充套件問題。
- 單一版本規則對組織效率的重要性出人意料。刪除送出地點或依賴內容的選擇可能會導致顯著的簡化。
- 在某些語言中，你可能會花一些精力來躲避這個問題，比如著色、單獨編譯、連結器隱藏等等技術方法。使這些方法正常工作的工作完全是徒勞的--你的軟體工程師並沒有生產任何東西，他們只是在解決技術債務。
- 以前的研究（DORA/State of DevOps/Accelerate）表明，基於幹線的開發是高績效開發組織的一個預測因素。長週期的開發分支不是一個好的預設計劃。
- 使用任何對你有意義的版本控制體系。如果你的組織想優先考慮為不同的專案建立獨立的倉庫，那麼取消儲存庫間的依賴關係/『基於頭部』/『基於主幹』可能仍然是明智的越來越多的VCS和建立系統設施允許您擁有小型、細粒度的儲存庫以及整個組織一致的『虛擬』頭/主幹概念。
