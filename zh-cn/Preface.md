## Preface 序言

This book is titled *Software Engineering at Google*. What precisely do we mean by software engineering? What distinguishes 『software engineering』 from 『programming』 or 『computer science』? And why would Google have a unique perspective to add to the corpus of previous software engineering literature written over the past 50 years?

本書的標題是*《谷歌的軟體工程》*。我們對軟體工程的確切定義是什麼？軟體工程 "與 "程式設計 "或 "電腦科學 "的區別是什麼？為什麼谷歌在過去50年的軟體工程文獻庫中會有那些獨特的視角？

The terms 『programming』 and 『software engineering』 have been used interchangeably for quite some time in our industry, although each term has a different emphasis and different implications. University students tend to study computer science and get jobs writing code as 『programmers.』

在我們的業界，"程式設計 "和 "軟體工程 "這兩個術語已經被交替使用了相當長的時間，儘管每個術語都有不同的重點和不同的含義。大學生傾向於學習電腦科學，並以作為"工程師 『的身份進行寫程式碼的工作。

『Software engineering,』 however, sounds more serious, as if it implies the application of some theoretical knowledge to build something real and precise. Mechanical engineers, civil engineers, aeronautical engineers, and those in other engineering disciplines all practice engineering. They all work in the real world and use the application of their theoretical knowledge to create something real. Software engineers also create 『something real,』 though it is less tangible than the things other engineers create.

然而，"軟體工程 "聽起來更加嚴肅，似乎它意味著應用理論知識來建立真實和精確的東西。機械工程師、土木工程師、航空工程師和其他工程學科的人都在進行工程實踐。他們都在現實世界中工作，運用他們的理論知識來創造一些真實的東西。軟體工程師也創造 "真實的東西"，儘管它沒有像其他工程師創造的東西那麼有形。

Unlike those more established engineering professions, current software engineering theory or practice is not nearly as rigorous. Aeronautical engineers must follow rigid guidelines and practices, because errors in their calculations can cause real damage; programming, on the whole, has traditionally not followed such rigorous practices. But, as software becomes more integrated into our lives, we must adopt and rely on more rigorous engineering methods. We hope this book helps others see a path toward more reliable software practices.

與那些更成熟的工程專業不同，目前的軟體工程理論或實踐方法還沒有那麼嚴格。航空工程師必須遵循嚴格的準則和實踐，因為他們的計算錯誤會造成真正的損失；而程式設計，總體來說，傳統上沒有遵循這樣嚴格的實踐。但是，隨著軟體越來越多地融入我們的生活，我們必須採用並依賴更嚴格的工程方法。我們希望這本書能幫助其他人看到一條通往更可靠的軟體實踐的道路。

### Programming Over Time 隨時間變化的程式設計

We propose that 『software engineering』 encompasses not just the act of writing code, but all of the tools and processes an organization uses to build and maintain that code over time. What practices can a software organization introduce that will best keep its code valuable over the long term? How can engineers make a codebase more sustainable and the software engineering discipline itself more rigorous? We don」t have fundamental answers to these questions, but we hope that Google」s collective experience over the past two decades illuminates possible paths toward finding those answers.

我們建議，"軟體工程 "不僅包括編寫程式碼的行為，還包括一個組織用來長期建立和隨時間維護程式碼的所有工具和流程。一個軟體組織可以採用哪些做法來使其程式碼長期保持最佳價值？工程師們如何才能使程式碼庫更具有可持續性，並使軟體工程學科本身更加嚴格？我們沒有這些問題的最終答案，但我們希望谷歌在過去20年的集體經驗能夠為尋找這些答案的提供可能。

One key insight we share in this book is that software engineering can be thought of as 『programming integrated over time.』 What practices can we introduce to our code to make it *sustainable*—able to react to necessary change—over its life cycle, from conception to introduction to maintenance to deprecation?

我們在本書中分享的一個關鍵觀點是，軟體工程可以被認為是 "隨著時間推移而整合的程式設計"。我們可以在我們的程式碼中引入哪些實踐，使其*可持續*——能夠對必要的變化做出反應——在其生命週期中，從設計到引入到維護到廢棄？

The book emphasizes three fundamental principles that we feel software organizations should keep in mind when designing, architecting, and writing their code:  
*Time* *and* *Change*
	How code will need to adapt over the length of its life
*Scale* *and Growth*
	How an organization will need to adapt as it evolves
*Trade-offs and Costs*
	How an organization makes decisions, based on the lessons of Time and Change and Scale and Growth

本書強調了三個基本原則，我們認為軟體組織在設計、架構和編寫程式碼時應該牢記這些原則：
*時間和變化*
	​程式碼如何在其生命週期內進行適配。
*規模和增長*
	​一個組織如何適應它的發展過程。
*權衡和成本*
	​一個組織如何根據時間和變化以及規模和增長的經驗教訓做出決策。

Throughout the chapters, we have tried to tie back to these themes and point out ways in which such principles affect engineering practices and allow them to be sustainable. (See [Chapter 1 ](#_bookmark3)for a full discussion.)

在整個章節中，我們都嘗試與這些主題聯絡起來，並指出這些原則如何影響工程實踐並使其可持續。(見[第1章](#_bookmark3)的全面討論)。

### Google」s Perspective 谷歌的視角

Google has a unique perspective on the growth and evolution of a sustainable soft‐ ware ecosystem, stemming from our scale and longevity. We hope that the lessons we have learned will be useful as your organization evolves and embraces more sustainable practices.

谷歌對可持續軟體生態系統的發展和演變有著獨特的視角，這源於我們的規模和壽命。我們希望在你的組織發展和採用更多的可持續發展的做法時，我們學到的經驗將能對你有幫助。

We」ve divided the topics in this book into three main aspects of Google」s software engineering landscape:
- Culture
- Processes
- Tools

我們將本書的主題分為谷歌軟體工程領域的三個主要方面：
- 文化
- 流程
- 工具

Google」s culture is unique, but the lessons we have learned in developing our engineering culture are widely applicable. Our chapters on Culture ([Part II](#_bookmark100)) emphasize the collective nature of a software development enterprise, that the development of software is a team effort, and that proper cultural principles are essential for an organization to grow and remain healthy.

谷歌的文化是獨一無二的，但我們在發展工程文化中所獲得的經驗是廣泛適用的。我們關於文化的章節（[第二部分](#_bookmark100)）強調了軟體開發企業的集體性，軟體開發是一項團隊工作，正確的文化原則對於一個組織的成長和保持健康至關重要。

The techniques outlined in our Processes chapters ([Part III](#_bookmark579)) are familiar to most soft‐ ware engineers, but Google」s large size and long-lived codebase provides a more complete stress test for developing best practices. Within those chapters, we have tried to emphasize what we have found to work over time and at scale as well as identify areas where we don」t yet have satisfying answers.

在我們的流程章節（[第三部分](#_bookmark579)）中概述的技術是大多數軟體工程師所熟悉的，但谷歌的龐大規模和長期的程式碼庫為開發最佳實踐提供了一個更完整的壓力測試。在這些章節中，我們強調我們發現隨著時間的推移和規模的擴大，什麼是有效的，以及確定我們還沒有滿意的答案的領域。

Finally, our Tools chapters ([Part IV](#_bookmark1363)) illustrate how we leverage our investments in tooling infrastructure to provide benefits to our codebase as it both grows and ages. In some cases, these tools are specific to Google, though we point out open source or third-party alternatives where applicable. We expect that these basic insights apply to most engineering organizations.

最後，我們的工具章節（[第四部分](#_bookmark1363)）說明了我們如何利用對工具基礎設施的投入來最佳化程式碼庫，因為它既增長又腐化。在某些情況下，這些工具是谷歌特有的，儘管我們在適當的地方指出了開源或第三方的替代品。我們希望這些基本的見解適用於大多數工程組織。

The culture, processes, and tools outlined in this book describe the lessons that a typical software engineer hopefully learns on the job. Google certainly doesn」t have a monopoly on good advice, and our experiences presented here are not intended to dictate what your organization should do. This book is our perspective, but we hope you will find it useful, either by adopting these lessons directly or by using them as a starting point when considering your own practices, specialized for your own problem domain.

本書中描寫的文化、流程和工具是大多數的軟體工程師希望在工作中使用的內容。谷歌當然不會獨斷好建議，我們在這裡介紹的經驗並不是要規定你的組織應當這麼做。本書是我們的觀點，但我們希望你會發現它是有用的，可以直接採用這些經驗，也可以在考慮自己的實踐時把它們作為一個起點，專門用於解決自己的領域問題。

Neither is this book intended to be a sermon. Google itself still imperfectly applies many of the concepts within these pages. The lessons that we have learned, we learned through our failures: we still make mistakes, implement imperfect solutions, and need to iterate toward improvement. Yet the sheer size of Google」s engineering organization ensures that there is a diversity of solutions for every problem. We hope that this book contains the best of that group.

本書也不打算成為一本佈道書。谷歌自身仍在不完善地應用這些書中的許多理念。我們從失敗中吸收了教訓：我們仍然會犯錯誤，採用不完美的解決方案，還需要迭代改進。然而，谷歌工程組織的龐大規模確定了每個問題都有多樣化的解決方案。我們希望這本書包含了這群人中最好的方案。

### What This Book Isn」t 本書不適用於哪些

This book is not meant to cover software design, a discipline that requires its own book (and for which much content already exists). Although there is some code in this book for illustrative purposes, the principles are language neutral, and there is little actual 『programming』 advice within these chapters. As a result, this text doesn」t cover many important issues in software development: project management, API design, security hardening, internationalization, user interface frameworks, or other language-specific concerns. Their omission in this book does not imply their lack of importance. Instead, we choose not to cover them here knowing that we could not provide the treatment they deserve. We have tried to make the discussions in this book more about engineering and less about programming.

本書並不是要涵蓋軟體設計，這門學科有自己的書（而且已經有很多型別的書）。雖然書中有一些程式碼用於說明問題，但原則是語言無關的，而且這些章節中幾乎沒有實際的 "程式設計 "建議。因此，本書沒有涉及軟體開發中的許多重要問題：專案管理、API設計、安全加固、國際化、使用者介面框架或其他特定程式語言問題。本書對這些問題的忽略並不意味著它們不重要。相反，我們選擇不在這裡涉及它們，因為我們知道我們無法提供它們應有的內容。我們試圖使本書的討論更多的關於工程領域，而不是關於程式設計領域。

### Parting Remarks 臨別贈言

This text has been a labor of love on behalf of all who have contributed, and we hope that you receive it as it is given: as a window into how a large software engineering organization builds its products. We also hope that it is one of many voices that helps move our industry to adopt more forward-thinking and sustainable practices. Most important, we further hope that you enjoy reading it and can adopt some of its lessons to your own concerns.

這篇文章是所有貢獻者的心血結晶，我們希望你能虛心地接受它：作為瞭解一個大型軟體工程組織如何建立其產品的視窗。我們還希望它是有助於推動我們的行業採用更具前瞻性和可持續實踐的眾多聲音之一。最重要的是，我們更希望你喜歡它，並能將其中的一些經驗用於你的工作。



*— Tom Manshreck*





