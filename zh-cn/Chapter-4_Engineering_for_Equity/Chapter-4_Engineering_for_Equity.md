
**CHAPTER 4**

# Engineering for Equity

# 第四章 公平工程

            Written by Demma Rodriguez Edited by Riona MacNamara

In earlier chapters, we’ve explored the contrast between programming as the production of code that addresses the problem of the moment, and software engineering as the broader application of code, tools, policies, and processes to a dynamic and ambiguous problem that can span decades or even lifetimes. In this chapter, we’ll discuss the unique responsibilities of an engineer when designing products for a broad base of users. Further, we evaluate how an organization, by embracing diversity, can design systems that work for everyone, and avoid perpetuating harm against our users.

在前幾章中，我們已經探討了程式設計與軟體工程之間的對比，前者是解決當下問題的程式碼生產，後者則是對程式碼、工具、策略和流程的更廣泛的應用，以解決可能跨越幾十年甚至一生的動態和模糊的問題。在本章中，我們將討論工程師在為眾多使用者設計產品時的獨特責任。此外，我們還將評估一個組織如何透過擁抱多樣性來設計適合每個人的系統，並避免對我們的使用者造成永久性的傷害。

As new as the field of software engineering is, we’re newer still at understanding the impact it has on underrepresented people and diverse societies. We did not write this chapter because we know all the answers. We do not. In fact, understanding how to engineer products that empower and respect all our users is still something Google is learning to do. We have had many public failures in protecting our most vulnerable users, and so we are writing this chapter because the path forward to more equitable products begins with evaluating our own failures and encouraging growth.

儘管軟體工程領域是個全新領域，但我們在瞭解它對代表性不足的群體和多元化社會的影響方面還比較淺。我們寫這一章並不是因為我們知道所有的答案。我們不知道。事實上，瞭解如何設計出能夠賦予所有使用者權益並尊重所有使用者的產品仍然是谷歌正在學習做的事情。在保護我們最弱勢的使用者方面，我們有很多公開的失敗產品，所以我們寫這一章是因為通往更平等的產品的道路始於評估我們自己的失敗和鼓勵成長。

We are also writing this chapter because of the increasing imbalance of power between those who make development decisions that impact the world and those who simply must accept and live with those decisions that sometimes disadvantage already marginalized communities globally. It is important to share and reflect on what we’ve learned so far with the next generation of software engineers. It is even more important that we help influence the next generation of engineers to be better than we are today.

我們之所以要寫這一章，也是因為在那些做出影響世界發展的人和那些只能選擇接受並忍受這些決定的人之間，力量越來越不平衡，這些決定有時使全球已經處於邊緣地位的社會處於不利地位。與下一代軟體工程師分享和反思我們迄今所學到的知識是很重要的。更重要的是，我們幫助影響下一代工程師，使他們比我們今天做得更好。

Just picking up this book means that you likely aspire to be an exceptional engineer. You want to solve problems. You aspire to build products that drive positive outcomes for the broadest base of people, including people who are the most difficult to reach. To do this, you will need to consider how the tools you build will be leveraged to change the trajectory of humanity, hopefully for the better.

只要拿起這本書，就意味著你可能立志成為一名出色的工程師。你想解決問題。你渴望建造產品，為最廣泛的人群，包括最難接觸的人，打造一個能帶來積極成果的產品。要做到這一點，你需要考慮如何利用你建造的工具來改變人類的軌跡，希望是為了獲得更好的發展。

## Bias Is the Default 偏見是預設的

When engineers do not focus on users of different nationalities, ethnicities, races, genders, ages, socioeconomic statuses, abilities, and belief systems, even the most talented staff will inadvertently fail their users. Such failures are often unintentional; all people have certain biases, and social scientists have recognized over the past several decades that most people exhibit unconscious bias, enforcing and promulgating existing stereotypes. Unconscious bias is insidious and often more difficult to mitigate than intentional acts of exclusion. Even when we want to do the right thing, we might not recognize our own biases. By the same token, our organizations must also recognize that such bias exists and work to address it in their workforces, product development, and user outreach.

當工程師不關注不同國籍、民族、種族、性別、年齡、社會經濟地位、能力和信仰體系的使用者時，即使是最優秀的工程師也會在不經意間讓使用者失望。這種失敗往往是無意的；所有的人都存在一定的偏見，社會科學家在過去幾十年中已經認識到，大多數人都表現出無意識的偏見，強迫和傳播存在的刻板印象。無意識的偏見是隱藏的，往往比有意的排斥行為更難改正。即使我們想做正確的事，我們也可能意識不到自己的偏見。同樣，我們的組織也必須認識到這種偏見的存在，並努力在員工隊伍、產品開發和使用者推廣中解決這一問題。

Because of bias, Google has at times failed to represent users equitably within their products, with launches over the past several years that did not focus enough on underrepresented groups. Many users attribute our lack of awareness in these cases to the fact that our engineering population is mostly male, mostly White or Asian, and certainly not representative of all the communities that use our products. The lack of representation of such users in our workforce[^1] means that we often do not have the requisite diversity to understand how the use of our products can affect underrepresented or vulnerable users.

由於偏見，谷歌有時未能在其產品中公平地代表使用者，在過去幾年中推出的產品沒有足夠關注代表性不足的群體。許多使用者將我們在這些情況下缺乏意識歸咎於這樣一個事實，即我們的工程人員大多數是男性，大多數是白人或亞洲人，當然不能代表所有使用我們產品的人群。這類別使用者在我們的員工隊伍中缺乏代表性，這意味著我們往往不具備必要的多樣性，無法理解使用我們的產品會如何影響代表性不足或弱勢的使用者。

------

#### Case Study: Google Misses the Mark on Racial Inclusion  案例研究：谷歌在種族包容方面的失誤

In 2015, software engineer Jacky Alciné pointed out[^2] that the image recognition algorithms in Google Photos were classifying his black friends as “gorillas.” Google was slow to respond to these mistakes and incomplete in addressing them.

2015年，軟體工程師Jacky Alciné指出，谷歌照片中的影象識別演算法將他的黑人朋友歸為 "大猩猩"。谷歌對這些錯誤的反應很慢，解決起來也不徹底。

What caused such a monumental failure? Several things:
- Image recognition algorithms depend on being supplied a “proper” (often meaning “complete”) dataset. The photo data fed into Google’s image recognition algorithm was clearly incomplete. In short, the data did not represent the population.
- Google itself (and the tech industry in general) did not (and does not) have much black representation,[^3] and that affects decisions subjective in the design of such algorithms and the collection of such datasets. The unconscious bias of the organization itself likely led to a more representative product being left on the table.
- Google’s target market for image recognition did not adequately include such underrepresented groups. Google’s tests did not catch these mistakes; as a result, our users did, which both embarrassed Google and harmed our users.

是什麼導致了這樣一個巨大的失誤？有幾件事：
- 影象識別演算法取決於是否提供了一個 "適當的"（通常意味著 "完整的"）資料集。送入谷歌影象識別演算法的照片資料顯然是不完整的。簡而言之，這些資料並不代表所有人口。
- 谷歌本身（以及整個科技行業）沒有（也沒有）很多黑人代表，這影響了設計這種演算法和收集這種資料集的主觀決定。組織本身無意識的偏見很可能導致更具代表性的產品被擱置。
- 谷歌的影象識別目標市場並沒有充分包括這種代表性不足的群體。谷歌的測試沒有發現這些錯誤；結果是我們的使用者發現了，這既讓谷歌感到尷尬，也傷害了我們的使用者。

As late as 2018, Google still had not adequately addressed the underlying problem.[^4]

直到2018年，谷歌仍然沒有徹底地解決這些潛在的問題。

------

In this example, our product was inadequately designed and executed, failing to properly consider all racial groups, and as a result, failed our users and caused Google bad press. Other technology suffers from similar failures: autocomplete can return offensive or racist results. Google’s Ad system could be manipulated to show racist or offensive ads. YouTube might not catch hate speech, though it is technically outlawed on that platform.

在這個例子中，我們的產品設計和執行不當，未能適當考慮到所有的種族群體，結果是辜負了我們的使用者，給谷歌帶來了惡劣的影響。其他技術也有類似的失誤：自動完成自動完成可以返回攻擊性或種族主義的結果。谷歌的廣告系統可以被操縱來顯示種族主義或攻擊性廣告。YouTube可能捕獲仇恨言論，儘管從技術上講，它在該平臺上是非法的。

In all of these cases, the technology itself is not really to blame. Autocomplete, for example, was not designed to target users or to discriminate. But it was also not resilient enough in its design to exclude discriminatory language that is considered hate speech. As a result, the algorithm returned results that caused harm to our users. The harm to Google itself should also be obvious: reduced user trust and engagement with the company. For example, Black, Latinx, and Jewish applicants could lose faith in Google as a platform or even as an inclusive environment itself, therefore undermining Google’s goal of improving representation in hiring.

在所有這些情況下，技術本身並不是真正的罪魁禍首。例如，自動完成自動完成的設計目的不是為了針對使用者或進行歧視。但它的設計也沒有足夠的靈活來排除被認為是仇恨言論的歧視性語言。結果，該演算法返回的結果對我們的使用者造成了傷害。對谷歌本身的損害也應該是顯而易見的：使用者對該公司的信任和參與度降低。例如，黑人、拉美人和猶太人的申請者可能會對谷歌這個平臺甚至其本身的包容性環境失去信心，因此破壞了谷歌改善招聘中的人群目標。

How could this happen? After all, Google hires technologists with impeccable education and/or professional experience—exceptional programmers who write the best code and test their work. “Build for everyone” is a Google brand statement, but the truth is that we still have a long way to go before we can claim that we do. One way to address these problems is to help the software engineering organization itself look like the populations for whom we build products.

這怎麼會發生呢？畢竟，谷歌僱用的技術專家擁有無可挑剔的教育和/或專業經驗——卓越的程式設計師，他們編寫最好的程式碼並測試他們的功能。"為每個人而建 "是谷歌的品牌宣言，但事實是，在宣稱我們做到這一點之前，我們仍有很長的路要走。解決這些問題的方法之一是幫助軟體工程組織本身變得像我們為其建造產品的人群。

> [^1]:    Google’s 2019 Diversity Report./
> 1 谷歌的2019年多樣性報告。
>
> [^2]:    @jackyalcine. 2015. “Google Photos, Y’all Fucked up. My Friend’s Not a Gorilla.” Twitter, June 29, 2015.https://twitter.com/jackyalcine/status/615329515909156865./
> 2 @jackyalcine. 2015. "谷歌照片，你們都搞砸了。我的朋友不是大猩猩"。Twitter，2015年6月29日。https://twitter.com/jackyalcine/status/615329515909156865
>
> [^3]:    Many reports in 2018–2019 pointed to a lack of diversity across tech. Some notables include the National Center for Women & Information Technology, and Diversity in Tech./
> 3  2018-2019年的許多報告指出，整個科技界缺乏多樣性。一些著名的報告包括國家婦女和資訊科技中心，以及科技領域的多樣性。
>
> [^4]:    Tom Simonite, “When It Comes to Gorillas, Google Photos Remains Blind,” Wired, January 11, 2018./
> 4    Tom Simonite，"當涉及到大猩猩時，谷歌照片仍然是盲目的，"《連線》，2018年1月11日。

## Understanding the Need for Diversity 瞭解多樣性的必要性

At Google, we believe that being an exceptional engineer requires that you also focus on bringing diverse perspectives into product design and implementation. It also means that Googlers responsible for hiring or interviewing other engineers must contribute to building a more representative workforce. For example, if you interview other engineers for positions at your company, it is important to learn how biased outcomes happen in hiring. There are significant prerequisites for understanding how to anticipate harm and prevent it. To get to the point where we can build for everyone, we first must understand our representative populations. We need to encourage engineers to have a wider scope of educational training.

在谷歌，我們相信，作為一名出色的工程師，你還需要專注於將不同的視角引入打牌產品設計和實施中。這也意味著，負責招聘或面試其他工程師的谷歌人必須致力於打造更具代表性的團隊。例如，如果你為公司的職位面試其他工程師，瞭解招聘過程中的偏差結果是如何發生，這是很重要的。瞭解如何預測和預防傷害有重要的先決條件。為了達到我們能夠為每個人而建的目的，我們首先必須瞭解我們的代表人群。瞭解招聘過程中的偏差結果是如何發生的是很重要的。

The first order of business is to disrupt the notion that as a person with a computer science degree and/or work experience, you have all the skills you need to become an exceptional engineer. A computer science degree is often a necessary foundation. However, the degree alone (even when coupled with work experience) will not make you an engineer. It is also important to disrupt the idea that only people with computer science degrees can design and build products. Today, [most programmers do have a computer science degree](https://oreil.ly/2Bu0H); they are successful at building code, establishing theories of change, and applying methodologies for problem solving. However, as the aforementioned examples demonstrate, *this approach is insufficient for inclusive and* *equitable engineering*.

首要的任務是打破這樣的觀念：作為一個擁有電腦科學學位或且工作經驗的人，你擁有成為一名出色工程師所需的所有技能。電腦科學學位通常是一個必要的基礎。然而，單憑學位（即使再加上工作經驗）並不能使你成為一名工程師。打破只有擁有電腦科學學位的人才能設計和建造產品的想法也很重要。今天，大多數程式設計師確實擁有電腦科學學位；他們在建構程式碼、建立變化理論和應用解決問題的方法方面都很成功。然而，正如上述例子所表明的，*這種方法不足以實現包容性和公平工程*。

Engineers should begin by focusing all work within the framing of the complete ecosystem they seek to influence. At minimum, they need to understand the population demographics of their users. Engineers should focus on people who are different than themselves, especially people who might attempt to use their products to cause harm. The most difficult users to consider are those who are disenfranchised by the processes and the environment in which they access technology. To address this challenge, engineering teams need to be representative of their existing and future users. In the absence of diverse representation on engineering teams, individual engineers need to learn how to build for all users.

工程師應首先關注他們試圖影響的完整生態系統框架內的所有工作。至少，他們需要了解使用者的人群統計資料。工程師應該關注與自己不同的人，特別是那些試圖使用他們的產品而造成傷害的人。最難考慮的使用者是那些被他們獲取技術的過程和環境所剝奪了權益的人。為了應對這一挑戰，工程團隊需要代表其現有和未來的使用者。在工程團隊缺乏多元化代表的情況下，每個工程師需要學習如何為所有使用者建構。

## Building Multicultural Capacity 建構多元化能力

One mark of an exceptional engineer is the ability to understand how products can advantage and disadvantage different groups of human beings. Engineers are expected to have technical aptitude, but they should also have the *discernment* to know when to build something and when not to. Discernment includes building the capacity to identify and reject features or products that drive adverse outcomes. This is a lofty and difficult goal, because there is an enormous amount of individualism that goes into being a high-performing engineer. Yet to succeed, we must extend our focus beyond our own communities to the next billion users or to current users who might be disenfranchised or left behind by our products.

卓越的工程師的一個標誌是能夠理解產品如何對不同的人群益處和害處。工程師應該有技術能力，但他們也應該有*敏銳的判斷力*，知道什麼時候該造什麼，什麼時候不該造。判斷力包括建立識別和拒絕那些導致不良結果的功能或產品的能力。這是一個崇高而艱難的目標，因為要成為一名出色的工程師，需要有大量的個人主義。然而，想要成功，我們必須擴大我們的關注範圍，關注我們當前使用者之外的未來十億的使用者，哪怕是可能被我們的產品剝奪權利或遺棄的現有使用者。

Over time, you might build tools that billions of people use daily—tools that influence how people think about the value of human lives, tools that monitor human activity, and tools that capture and persist sensitive data, such as images of their children and loved ones, as well as other types of sensitive data. As an engineer, you might wield more power than you realize: the power to literally change society. It’s critical that on your journey to becoming an exceptional engineer, you understand the innate responsibility needed to exercise power without causing harm. The first step is to recognize the default state of your bias caused by many societal and educational factors. After you recognize this, you’ll be able to consider the often-forgotten use cases or users who can benefit or be harmed by the products you build.

隨著時間的推移，你可能會建立數十億人每天使用的工具——影響人們思考人類生命價值的工具，監測人類活動的工具，以及捕獲和永久儲存敏感資料的工具，如他們的孩子和親人的影象，以及其他型別的敏感資料。作為一名工程師，你可能掌握著比你意識到的更多的權力：真正改變社會的權力。至關重要的是，在你成為一名傑出的工程師的過程中，你必須理解在不造成傷害的情況下行使權力所需的內在責任，這一點至關重要。第一步是要認識到由許多社會和教育因素造成的你的偏見的預設狀態。在你認識到這一點之後，你就能考慮那些經常被遺忘的用例或使用者，他們可以從你製造的產品中獲益或受到傷害。

The industry continues to move forward, building new use cases for artificial intelligence (AI) and machine learning at an ever-increasing speed. To stay competitive, we drive toward scale and efficacy in building a high-talent engineering and technology workforce. Yet we need to pause and consider the fact that today, some people have the ability to design the future of technology and others do not. We need to understand whether the software systems we build will eliminate the potential for entire populations to experience shared prosperity and provide equal access to technology.

軟體行業持續發展，以不斷提高的速度為人工智慧（AI）和機器學習建立新的用例。為了保持競爭力，我們在建設高素質的工程和技術人才隊伍方面，朝著規模和效率的方向努力。然而，我們需要暫停並考慮這樣一個事實：今天，有些人有能力設計技術的未來，其他人卻沒有。我們需要了解我們建立的軟體系統是否會消除整個人口體驗共同繁榮的潛力，並提供平等獲得技術的機會。

Historically, companies faced with a decision between completing a strategic objective that drives market dominance and revenue and one that potentially slows momentum toward that goal have opted for speed and shareholder value. This tendency is exacerbated by the fact that many companies value individual performance and excellence, yet often fail to effectively drive accountability on product equity across all areas. Focusing on underrepresented users is a clear opportunity to promote equity. To continue to be competitive in the technology sector, we need to learn to engineer for global equity.

從歷史上看，公司在完成推動市場主導地位和收入的戰略目標和可能減緩實現這一目標勢頭的戰略目標之間，都選擇了速度和股東價值。許多公司重視個人的績效和卓越，但往往不能有效地推動各領域的產品公平的問責機制，這加劇了這種傾向。關注代表性不足的使用者顯然是促進公平的機會。為了在技術領域繼續保持競爭力，我們需要學習如何設計全球公平。

Today, we worry when companies design technology to scan, capture, and identify people walking down the street. We worry about privacy and how governments might use this information now and in the future. Yet most technologists do not have the requisite perspective of underrepresented groups to understand the impact of racial variance in facial recognition or to understand how applying AI can drive harmful and inaccurate results.

如今，當公司設計掃描、捕獲和識別街上行人的技術時，我們感到擔憂。我們擔心隱私問題以及政府現在和將來如何使用這些資訊。然而，大多數技術專家並不具備代表性不足群體的必要視角，無法理解種族差異對面部識別的影響，也無法理解應用人工智慧如何導致有害和不準確的結果。

Currently, AI-driven facial-recognition software continues to disadvantage people of color or ethnic minorities. Our research is not comprehensive enough and does not include a wide enough range of different skin tones. We cannot expect the output to be valid if both the training data and those creating the software represent only a small subsection of people. In those cases, we should be willing to delay development in favor of trying to get more complete and accurate data, and a more comprehensive and inclusive product.

目前，人工智慧驅動的面部識別軟體仍然對有色人種或少數族裔不利。我們的研究還不夠全面，沒有包括足夠多的膚色。如果訓練資料和建立軟體的人都只代表一小部分人，我們就不能指望輸出是有效的。在這種情況下，我們應該願意推遲開發，以獲得更完整、更準確的資料，以及更全面、更包容的產品。

Data science itself is challenging for humans to evaluate, however. Even when we do have representation, a training set can still be biased and produce invalid results. A study completed in 2016 found that more than 117 million American adults are in a law enforcement facial recognition database.[^5] Due to the disproportionate policing of Black communities and disparate outcomes in arrests, there could be racially biased error rates in utilizing such a database in facial recognition. Although the software is being developed and deployed at ever-increasing rates, the independent testing is not. To correct for this egregious misstep, we need to have the integrity to slow down and ensure that our inputs contain as little bias as possible. Google now offers statistical training within the context of AI to help ensure that datasets are not intrinsically biased.

然而，資料科學本身對人類的評估是具有挑戰性的。即使我們有表示，訓練集仍然可能有偏見，產生無效的結果。2016年完成的一項研究發現，執法部門的面部識別資料庫中有1.17億以上的美國成年人。由於黑人社群的警察比例過高，逮捕的結果也不盡相同，因此在面部識別中使用該資料庫可能存在種族偏見錯誤率。儘管該軟體的開發和部署速度不斷提高，但獨立測試卻並非如此。為了糾正這一令人震驚的錯誤，我們需要有誠信，放慢腳步，確保我們的輸入儘可能不包含偏見。谷歌現在在人工智慧的範圍內提供統計培訓，以幫助確保資料集沒有內在的偏見。

Therefore, shifting the focus of your industry experience to include more comprehensive, multicultural, race and gender studies education is not only your responsibility, but also the responsibility of your employer. Technology companies must ensure that their employees are continually receiving professional development and that this development is comprehensive and multidisciplinary. The requirement is not that one individual take it upon themselves to learn about other cultures or other demographics alone. Change requires that each of us, individually or as leaders of teams, invest in continuous professional development that builds not just our software development and leadership skills, but also our capacity to understand the diverse experiences throughout humanity.

因此，將你的行業經驗的重點轉移到更全面的、多文化的、種族和性別研究的教育，不僅是你的責任，也是你僱主的責任。科技公司必須確保他們的員工不斷接受專業發展，而且這種發展是全面和多學科的。要求不是個體獨自承擔起學習其他文化或其他人口統計學的任務。變革要求我們每個人，無論是個人還是團隊的領導者，都要投資於持續的專業發展，不僅要培養我們的軟體開發和領導技能，還要培養我們理解全人類不同經驗的能力。

> [^5]:    Stephen Gaines and Sara Williams. “The Perpetual Lineup: Unregulated Police Face Recognition in America.”/  
> 5    斯蒂芬·蓋恩斯和莎拉·威廉姆斯。“永遠的陣容：美國不受監管的警察面孔識別。”
喬治敦法律學院隱私與技術中心，2016年10月18日。


## Making Diversity Actionable 讓多樣性成為現實

Systemic equity and fairness are attainable if we are willing to accept that we are all accountable for the systemic discrimination we see in the technology sector. We are accountable for the failures in the system. Deferring or abstracting away personal accountability is ineffective, and depending on your role, it could be irresponsible. It is also irresponsible to fully attribute dynamics at your specific company or within your team to the larger societal issues that contribute to inequity. A favorite line among diversity proponents and detractors alike goes something like this: “We are working hard to fix (insert systemic discrimination topic), but accountability is hard. How do we combat (insert hundreds of years) of historical discrimination?” This line of inquiry is a detour to a more philosophical or academic conversation and away from focused efforts to improve work conditions or outcomes. Part of building multicultural capacity requires a more comprehensive understanding of how systems of inequality in society impact the workplace, especially in the technology sector.

如果我們願意接受我們需要對我們在技術部門看到的系統歧視負責，那麼系統的公平和公正是可以實現的。我們要對系統的故障負責。推遲或抽離個人責任是無效的，而且根據你的角色，這可能是不負責任的。將特定公司或團隊的動態完全歸因於導致不平等的更大社會問題也是不負責任的。多樣性支持者和反對者中最喜歡的一句話是這樣的。"我們正在努力解決（加入系統歧視的話題），但問責是很難的。我們如何打擊（加入幾百年來的）歷史歧視？" 這條調查路線是一條通往哲學或學術對話的迂迴之路，與改善工作條件或成果的專注努力相去甚遠。建設多元文化能力的一部分需要更全面地瞭解社會中的不平等制度如何影響工作場所，特別是在技術部門。

If you are an engineering manager working on hiring more people from underrepresented groups, deferring to the historical impact of discrimination in the world is a useful academic exercise. However, it is critical to move beyond the academic conversation to a focus on quantifiable and actionable steps that you can take to drive equity and fairness. For example, as a hiring software engineer manager, you’re accountable for ensuring that your candidate slates are balanced. Are there women or other underrepresented groups in the pool of candidates’ reviews? After you hire someone, what opportunities for growth have you provided, and is the distribution of opportunities equitable? Every technology lead or software engineering manager has the means to augment equity on their teams. It is important that we acknowledge that, although there are significant systemic challenges, we are all part of the system. It is our problem to fix.

如果你是一名工程經理，致力於僱用更多來自代表性不足的群體的人，推崇世界上歧視的歷史影響是一項有益的學術活動。然而，關鍵是要超越學術交流，把重點放在可量化和可操作的步驟上，以推動公平和公正。例如，作為招聘軟體工程師經理，你有責任確保你的候選人名單是均衡的。在候選人的審查中是否有女性或其他代表性不足的群體？僱傭員工後，你提供了哪些成長機會，機會分配是否公平？每個技術領導或軟體工程經理都有辦法在他們的團隊中增加平等。重要的是，我們要承認，儘管存在著重大的系統性挑戰，但我們都是這個系統的一部分。這是我們要解決的問題。

## Reject Singular Approaches 摒棄單一方法

We cannot perpetuate solutions that present a single philosophy or methodology for fixing inequity in the technology sector. Our problems are complex and multifactorial. Therefore, we must disrupt singular approaches to advancing representation in the workplace, even if they are promoted by people we admire or who have institutional power.

我們不能讓那些提出單一理念或方法來解決技術部門不公平問題的解決方案永久化。我們的問題是複雜和多因素的。因此，我們必須打破推進工作場所代表性的單一方法，即使這些方法是由我們敬佩的人或擁有機構權力的人推動的。

One singular narrative held dear in the technology industry is that lack of representation in the workforce can be addressed solely by fixing the hiring pipelines. Yes, that is a fundamental step, but that is not the immediate issue we need to fix. We need to recognize systemic inequity in progression and retention while simultaneously focusing on more representative hiring and educational disparities across lines of race, gender, and socioeconomic and immigration status, for example.

在科技行業中，有一種單一的說法是，勞動力中缺乏代表性的問題可以只通過解決招聘管道來解決。是的，這是一個基本步驟，但這並不是我們需要解決的緊迫問題。我們需要認識到在晉升和留任方面的系統不平等，同時關注更具代表性的招聘和教育差異，例如種族、性別、社會經濟和移民狀況。

In the technology industry, many people from underrepresented groups are passed over daily for opportunities and advancement. Attrition among Black+ Google employees outpaces attrition from all other groups and confounds progress on representation goals. If we want to drive change and increase representation, we need to evaluate whether we’re creating an ecosystem in which all aspiring engineers and other technology professionals can thrive.

在科技行業，許多來自代表性不足的群體的人每天都被排除在機會和晉升之外。谷歌黑人員工的流失率超過了所有其他群體的流失率，並影響了代表目標的實現。如果我們想推動變革並提高代表性，我們需要評估我們是否正在創造一個所有有抱負的工程師和其他技術專業人員都能茁壯成長的生態系統。

Fully understanding an entire problem space is critical to determining how to fix it. This holds true for everything from a critical data migration to the hiring of a representative workforce. For example, if you are an engineering manager who wants to hire more women, don’t just focus on building a pipeline. Focus on other aspects of the hiring, retention, and progression ecosystem and how inclusive it might or might not be to women. Consider whether your recruiters are demonstrating the ability to identify strong candidates who are women as well as men. If you manage a diverse engineering team, focus on psychological safety and invest in increasing multicultural capacity on the team so that new team members feel welcome.

充分了解整個問題空間對於確定如何解決它至關重要。這適用於從關鍵資料遷移到僱傭代表性員工的所有方面。例如，如果你是一個想僱用更多女性的工程經理，不要只關注單個方面建設。關注招聘、保留和晉升生態系統的其他方面，以及它對女性的包容性。考慮一下你的招聘人員是否展示了識別女性和男性候選人的能力。如果你管理一個多元化的工程團隊，請關注心理安全，並投入於增加團隊的多元文化能力，使新的團隊成員感到受歡迎。

A common methodology today is to build for the majority use case first, leaving improvements and features that address edge cases for later. But this approach is flawed; it gives users who are already advantaged in access to technology a head start, which increases inequity. Relegating the consideration of all user groups to the point when design has been nearly completed is to lower the bar of what it means to be an excellent engineer. Instead, by building in inclusive design from the start and raising development standards for development to make tools delightful and accessible for people who struggle to access technology, we enhance the experience for all users.

如今，一種常見的方法是首先為大多數用例建構，將解決邊緣用例的改進和特性留待以後使用。但這種方法是有缺陷的；它讓那些在獲取技術方面已經有優勢的使用者搶先一步，這增加了不平等。把對所有使用者群體的考慮放在設計即將完成的時候，就是降低成為一名優秀工程師的標準。相反，透過從一開始就採用包容性設計，提高開發標準，讓那些難以獲得技術的人能夠輕鬆地使用工具，我們增強了所有使用者的體驗。

Designing for the user who is least like you is not just wise, it’s a best practice. There are pragmatic and immediate next steps that all technologists, regardless of domain, should consider when developing products that avoid disadvantaging or underrepresenting users. It begins with more comprehensive user-experience research. This research should be done with user groups that are multilingual and multicultural and that span multiple countries, socioeconomic class, abilities, and age ranges. Focus on the most difficult or least represented use case first.

為最不喜歡你的使用者設計不僅是明智的，而且是最佳實踐。所有的技術專家，無論在哪個領域，在開發產品時都應該考慮一些實用的和直接的步驟，以避免對使用者造成不利影響或代表不足。它從更全面的使用者體驗研究開始。這項研究應該針對多語言、多文化、跨多個國家、社會經濟階層、能力和年齡範圍的使用者群體進行。首先關注最困難或最不典型的用例。

## Challenge Established Processes 挑戰既定流程

Challenging yourself to build more equitable systems goes beyond designing more inclusive product specifications. Building equitable systems sometimes means challenging established processes that drive invalid results.

挑戰自己以建立更公平的系統，不僅僅是設計更具包容性的產品規格。建立公平系統有時意味著挑戰那些推動無效結果的既定流程。

Consider a recent case evaluated for equity implications. At Google, several engineering teams worked to build a global hiring requisition system. The system supports both external hiring and internal mobility. The engineers and product managers involved did a great job of listening to the requests of what they considered to be their core user group: recruiters. The recruiters were focused on minimizing wasted time for hiring managers and applicants, and they presented the development team with use cases focused on scale and efficiency for those people. To drive efficiency, the recruiters asked the engineering team to include a feature that would highlight performance ratings—specifically lower ratings—to the hiring manager and recruiter as soon as an internal transfer expressed interest in a job.

考慮一下最近一個被評估為對公平有影響的案例。在谷歌，幾個工程團隊致力於建立一個全球招聘申請系統。該系統同時支援外部招聘和內部流動。參與的工程師和產品經理在傾聽他們認為是他們的核心使用者群體的請求方面做得很好：招聘人員。招聘人員專注於最大限度地減少招聘經理和申請人的時間浪費，他們向開發團隊提出了專注於這些人的規模和效率的案例。為了提高效率，招聘人員要求工程團隊加入一項功能，在內部調動人員表示對某項工作感興趣時，該功能將突出績效評級，特別是向招聘經理和招聘人員提供較低的評級。

On its face, expediting the evaluation process and helping job seekers save time is a great goal. So where is the potential equity concern? The following equity questions were raised:
- Are developmental assessments a predictive measure of performance?
- Are the performance assessments being presented to prospective managers free of individual bias?
- •Are performance assessment scores standardized across organizations?

從表面上看，加快評估過程和幫助求職者節省時間是一個偉大的目標。那麼，潛在的公平問題在哪裡？以下是提出的公平問題。
- 發展評估是否是績效的預測指標？
- 向潛在經理提交的績效評估是否沒有個人偏見？
- 績效評估的分數在不同的組織中是標準化的嗎？

If the answer to any of these questions is “no,” presenting performance ratings could still drive inequitable, and therefore invalid, results.

如果這些問題的答案都是 "否"，呈現績效評級仍然可能導致不公平，因此是無效的結果。

When an exceptional engineer questioned whether past performance was in fact predictive of future performance, the reviewing team decided to conduct a thorough review. In the end, it was determined that candidates who had received a poor performance rating were likely to overcome the poor rating if they found a new team. In fact, they were just as likely to receive a satisfactory or exemplary performance rating as candidates who had never received a poor rating. In short, performance ratings are indicative only of how a person is performing in their given role at the time they are being evaluated. Ratings, although an important way to measure performance during a specific period, are not predictive of future performance and should not be used to gauge readiness for a future role or qualify an internal candidate for a different team. (They can, however, be used to evaluate whether an employee is properly or improperly slotted on their current team; therefore, they can provide an opportunity to evaluate how to better support an internal candidate moving forward.)

當一位傑出的工程師質疑過去的業績是否真的能預測未來的業績時，審查小組決定進行一次徹底的審查。最後確定，曾經獲得不良業績評級的候選人如果找到一個新的團隊，就有可能克服較差的評級。事實上，他們獲得滿意或堪稱楷模績效評級的可能性與從未獲得過差評的候選人一樣。簡而言之，績效評級僅表示一個人在擔任指定角色時的表現。評級雖然是衡量特定時期績效的一種重要方式，但不能預測未來績效，不應用於衡量未來角色的準備情況或確定不同團隊的內部候選人。(然而，它們可以被用來評估一個員工在其當前團隊中的位置是否合適；因此，它們可以提供一個機會來評估如何更好地支援內部候選人發展。）

This analysis definitely took up significant project time, but the positive trade-off was a more equitable internal mobility process.

這一分析無疑佔用了大量的專案時間，但積極的權衡是一個更公平的內部流動過程。

## Values Versus Outcomes 價值觀與成果

Google has a strong track record of investing in hiring. As the previous example illustrates, we also continually evaluate our processes in order to improve equity and inclusion. More broadly, our core values are based on respect and an unwavering commitment to a diverse and inclusive workforce. Yet, year after year, we have also missed our mark on hiring a representative workforce that reflects our users around the globe. The struggle to improve our equitable outcomes persists despite the policies and programs in place to help support inclusion initiatives and promote excellence in hiring and progression. The failure point is not in the values, intentions, or investments of the company, but rather in the application of those policies at the implementation level.

谷歌在招聘方面有著良好的投入記錄。正如前面的例子所示，我們也在不斷評估我們的流程，以提高公平和包容。更廣泛地說，我們的核心價值觀是基於尊重、對多元化和包容性勞動力的堅定承諾。然而，一年又一年，我們在僱用一支反映我們全球使用者的代表性員工隊伍方面卻沒有達到目標。儘管制定了策略和計劃，以幫助支援包容倡議並促進招聘和晉升的卓越性，但改善公平結果的鬥爭依然存在。失敗點不在於公司的價值觀、意圖或投入，而在於這些策略在執行層面的應用。

Old habits are hard to break. The users you might be used to designing for today— the ones you are used to getting feedback from—might not be representative of all the users you need to reach. We see this play out frequently across all kinds of products, from wearables that do not work for women’s bodies to video-conferencing software that does not work well for people with darker skin tones.

舊習慣很難改掉。你今天可能習慣於為之設計的使用者——你習慣於從他們那裡獲得反饋——可能並不代表你需要接觸的所有使用者。我們看到這種情況經常發生在各種產品上，從不適合女性身體的可穿戴裝置到不適合深膚色人的視訊會議軟體。

So, what’s the way out?

1. Take a hard look in the mirror. At Google, we have the brand slogan, “Build For Everyone.” How can we build for everyone when we do not have a representative workforce or engagement model that centralizes community feedback first? We can’t. The truth is that we have at times very publicly failed to protect our most vulnerable users from racist, antisemitic, and homophobic content.
2. Don’t build for everyone. Build with everyone. We are not building for everyone yet. That work does not happen in a vacuum, and it certainly doesn’t happen when the technology is still not representative of the population as a whole. That said, we can’t pack up and go home. So how do we build for everyone? We build with our users. We need to engage our users across the spectrum of humanity and be intentional about putting the most vulnerable communities at the center of our design. They should not be an afterthought.
3. Design for the user who will have the most difficulty using your product. Building for those with additional challenges will make the product better for everyone. Another way of thinking about this is: don’t trade equity for short-term velocity.
4. Don’t assume equity; measure equity throughout your systems. Recognize that decision makers are also subject to bias and might be undereducated about the causes of inequity. You might not have the expertise to identify or measure the scope of an equity issue. Catering to a single userbase might mean disenfranchising another; these trade-offs can be difficult to spot and impossible to reverse. Partner with individuals or teams that are subject matter experts in diversity, equity, and inclusion.
5. Change is possible. The problems we’re facing with technology today, from surveillance to disinformation to online harassment, are genuinely overwhelming. We can’t solve these with the failed approaches of the past or with just the skills we already have. We need to change.

那麼，出路是什麼？

1. 認真照照鏡子。在谷歌，我們有一個品牌口號，"為每個人而建"。當我們沒有一個代表性的員工隊伍或首先集中社群反饋的參與模式時，我們如何為每個人建設？我們不能。事實是，我們有時在公開場合未能保護我們最脆弱的使用者免受種族主義、反猶太主義和恐同內容的侵害。
2. 不要為每個人而建。要與所有人一起共建。我們還沒有為每個人建設的能力。這項工作不會憑空實現，當技術仍然不能代表整個人口時，這項工作肯定不會發生。話雖如此，我們也不能打包回家。那麼，我們如何為每個人建立？我們與我們的使用者一起建設。我們需要讓全人類的使用者參與進來，並有意將最脆弱的群體置於我們設計的中心。他們不應該是事後的考慮物件。
3. 為那些在使用你的產品時遇到最大困難的使用者設計。為那些有額外挑戰的人設計將使產品對所有人都更好。另一種思考方式是：不要用公平來換取短期的速度。
4. 不要假設公平；**衡量整個系統的公平性**。認識到決策者也會有偏見，而且可能對不平等的原因認識不足。你可能不具備識別或衡量公平問題的範圍的專業知識。迎合單個使用者群可能意味著剝奪另一個使用者群的權利；這些權衡可能很難發現，也不可能逆轉。與作為多元化主題專家的個人或團隊合作，公平、平等和包容。
5. 改變是可能的。我們今天所面臨的技術問題，從監視到虛假資訊再到線上騷擾，確實是令人難以承受的。我們不能用過去失敗的方法或只用我們已有的技能來解決這些問題。我們需要改變。

## Stay Curious, Push Forward 保持好奇心，勇往直前

The path to equity is long and complex. However, we can and should transition from simply building tools and services to growing our understanding of how the products we engineer impact humanity. Challenging our education, influencing our teams and managers, and doing more comprehensive user research are all ways to make progress. Although change is uncomfortable and the path to high performance can be painful, it is possible through collaboration and creativity.

通往公平的道路是道阻且長。然而，我們可以也應該從簡單地建構工具和服務過渡到加深我們對我們設計的產品如何影響人類的理解。挑戰我們的教育，影響我們的團隊和管理者，以及做更全面的使用者研究，都是取得進展的方法。雖然改變是痛苦的，而且通向高績效的道路可能是痛苦的，但透過合作和創新，變革是可能的。

Lastly, as future exceptional engineers, we should focus first on the users most impacted by bias and discrimination. Together, we can work to accelerate progress by focusing on Continuous Improvement and owning our failures. Becoming an engineer is an involved and continual process. The goal is to make changes that push humanity forward without further disenfranchising the disadvantaged. As future exceptional engineers, we have faith that we can prevent future failures in the system.

最後，作為未來的傑出工程師，我們應該首先關注受偏見和歧視影響最大的使用者。透過共同努力，我們可以透過專注於持續改進和承認失敗來加速進步。成為一名工程師是一個複雜而持續的過程。目標是在不進一步剝奪弱勢群體權利的情況下，做出推動人類前進的變革。作為未來傑出的工程師，我們有信心能夠防止未來系統的失敗。

## Conclusion 總結

Developing software, and developing a software organization, is a team effort. As a software organization scales, it must respond and adequately design for its user base, which in the interconnected world of computing today involves everyone, locally and around the world. More effort must be made to make both the development teams that design software and the products that they produce reflect the values of such a diverse and encompassing set of users. And, if an engineering organization wants to scale, it cannot ignore underrepresented groups; not only do such engineers from these groups augment the organization itself, they provide unique and necessary perspectives for the design and implementation of software that is truly useful to the world at large.

開發軟體和開發軟體組織是一項團隊工作。隨著軟體組織規模的擴大，它必須對其使用者群做出響應並進行充分設計，在當今互聯的計算世界中，使用者群涉及到本地和世界各地的每個人。必須做出更多的努力，使設計軟體的開發團隊和他們生產的產品都能反映出這樣一個多樣化的、包含了所有使用者的價值觀。而且，如果一個工程組織想要擴大規模，它不能忽視代表性不足的群體；這些來自這些群體的工程師不僅能增強組織本身，還能為設計和實施對整個世界真正有用的軟體提供獨特而必要的視角。

## TL;DRs  內容提要

- Bias is the default.

- Diversity is necessary to design properly for a comprehensive user base.

- Inclusivity is critical not just to improving the hiring pipeline for underrepresented groups, but to providing a truly supportive work environment for all people.

- Product velocity must be evaluated against providing a product that is truly useful to all users. It’s better to slow down than to release a product that might cause harm to some users.

- 偏見是預設的。

- 多樣性是正確設計綜合使用者群所必需的。

- 包容性不僅對於改善代表不足的群體的招聘渠道至關重要，而且對於為所有人提供一個真正支援性的工作環境也至關重要。

- 產品速度必須根據提供對所有使用者真正有用的產品來評估。與其發佈一個可能對某些使用者造成傷害的產品，還不如放慢速度。
