## Foreword 序言

I have always been endlessly fascinated with the details of how Google does things. I have grilled my Googler friends for information about the way things really work inside of the company. How do they manage such a massive, monolithic code repository without falling over? How do tens of thousands of engineers successfully collaborate on thousands of projects? How do they maintain the quality of their systems?

我對谷歌做事的細節著迷不已。我也曾向在谷歌工作的朋友問詢谷歌內部如何運作。他們是如何管理如此龐大的單體程式碼函式庫而不出錯的？數以萬計的工程師是如何在數千個專案上成功協作的？他們是如何保持系統的品質的？

Working with former Googlers has only increased my curiosity. If you’ve ever worked with a former Google engineer (or “Xoogler,” as they’re sometimes called), you’ve no doubt heard the phrase “at Google we…” Coming out of Google into other companies seems to be a shocking experience, at least from the engineering side of things. As far as this outsider can tell, the systems and processes for writing code at Google must be among the best in the world, given both the scale of the company and how often peo‐ ple sing their praises.

與前谷歌員工一起共事，只會增加我的好奇心。如果你曾經與前谷歌工程師（或他們有時稱之為“Xoogler”）一起工作，你無疑聽到過這樣一句話："在谷歌我們......" 從谷歌出來進入其他公司已經是一個令人羨慕的經歷，至少從工程方面來說是這樣。就我這個局外人而言，考慮到公司的規模和員工對其的讚譽程度，谷歌公司編寫程式碼的系統和流程一定是世界上最好的之一。

In *Software Engineering at Google*, a set of Googlers (and some Xooglers) gives us a lengthy blueprint for many of the practices, tools, and even cultural elements that underlie software engineering at Google. It’s easy to overfocus on the amazing tools that Google has built to support writing code, and this book provides a lot of details about those tools. But it also goes beyond simply describing the tooling to give us the philosophy and processes that the teams at Google follow. These can be adapted to fit a variety of circumstances, whether or not you have the scale and tooling. To my delight, there are several chapters that go deep on various aspects of automated testing, a topic that continues to meet with too much resistance in our industry.

在*《Google的軟體工程》*中，一組Googlers（和一些Xooglers）為我們提供了谷歌軟體工程的許多實踐、工具甚至文化元素的詳細藍圖。我們很容易過度關注谷歌為支援編寫程式碼而建構的神奇工具，本書提供了很多關於這些工具的細節。本書不僅僅是簡單地描述工具，為我們提供谷歌團隊遵循的理念和流程。這些都可以適應各種情況，無論你是否有這樣的規模和工具。令我興奮的是，有幾個章節深入探討了自動化測試的各個方面，這個話題在我們的行業中仍然遇到太多的阻力。

The great thing about tech is that there is never only one way to do something. Instead, there is a series of trade-offs we all must make depending on the circumstances of our team and situation. What can we cheaply take from open source? What can our team build? What makes sense to support for our scale? When I was grilling my Googler friends, I wanted to hear about the world at the extreme end of scale: resource rich, in both talent and money, with high demands on the software being built. This anecdotal information gave me ideas on some options that I might not otherwise have considered.

技術的偉大之處在於，做一件事永遠不會只有一種方法。相反，有一系列的權衡，我們都必須根據我們的團隊和現狀來選擇。我們可以從開放原始碼中低成本地獲取什麼？我們的團隊可以建立什麼？對我們的規模來說，什麼是有意義的支援？當我在詢問我的Googler朋友時，我想聽聽處於規模之顛的世界：要錢有錢，要人有人，對正在建構的軟體要求很高。這些資訊給了我一些想法，這些想法可能是我沒有思考過的。

With this book, we’ve written down those options for everyone to read. Of course, Google is a unique company, and it would be foolish to assume that the right way to run your software engineering organization is to precisely copy their formula. Applied practically, this book will give you ideas on how things could be done, and a lot of information that you can use to bolster your arguments for adopting best practices like testing, knowledge sharing, and building collaborative teams.

透過這本書，我們把這些選擇寫下來供大家閱讀。當然，谷歌是一家獨一無二的公司，如果認為執行你的軟體工程組織的正確方法是精確地複製他們的模式，那就太愚蠢了。在實際應用中，這本書會給你提供關於如何做事情的想法，以及很多資訊，你可以用這些資訊來支援你採用最佳實踐的論據，如測試、知識共享和建立協作團隊。

You may never need to build Google yourself, and you may not even want to reach for the same techniques they apply in your organization. But if you aren’t familiar with the practices Google has developed, you’re missing a perspective on software engineering that comes from tens of thousands of engineers working collaboratively on software over the course of more than two decades. That knowledge is far too valuable to ignore.

你可能永遠不需要自己建立谷歌，你甚至可能不想在你的組織中使用他們所應用的技術。但是，如果你不熟悉谷歌開發的實踐，你就會錯過一個關於軟體工程的視角，這個視角來自於二十多年來數萬名工程師在軟體上的協作。這些知識太有價值了，不能忽視。

 

*— Camille Fournier* *Author,* The Manager’s Path