

**CHAPTER 12**

# Unit Testing

# 第十二章 單元測試

**Written by Erik Kuefler**

**Edited by Tom Manshreck**

The previous chapter introduced two of the main axes along which Google classifies tests: *size* and *scope*. To recap, size refers to the resources consumed by a test and what it is allowed to do, and scope refers to how much code a test is intended to validate. Though Google has clear definitions for test size, scope tends to be a little fuzzier. We use the term *unit test* to refer to tests of relatively narrow scope, such as of a single class or method. Unit tests are usually small in size, but this isn’t always the case.

上一章介紹了谷歌對測試進行分類的兩個主要軸線：*大小*和*範圍*。簡而言之，大小是指測試所消耗的資源和允許做的事情，範圍是指測試要驗證多少程式碼。雖然谷歌對測試規模有明確的定義，但範圍往往是比較模糊的。我們使用術語*單元測試*指的是範圍相對較窄的測試，如單個類別或方法的測試。單元測試通常是小規模的，但並不總是如此。

After preventing bugs, the most important purpose of a test is to improve engineers’ productivity. Compared to broader-scoped tests, unit tests have many properties that make them an excellent way to optimize productivity:

- They tend to be small according to Google’s definitions of test size. Small tests are fast and deterministic, allowing developers to run them frequently as part of their workflow and get immediate feedback.
- They tend to be easy to write at the same time as the code they’re testing, allowing engineers to focus their tests on the code they’re working on without having to set up and understand a larger system.
- They promote high levels of test coverage because they are quick and easy to write. High test coverage allows engineers to make changes with confidence that they aren’t breaking anything.
- They tend to make it easy to understand what’s wrong when they fail because each test is conceptually simple and focused on a particular part of the system.
- They can serve as documentation and examples, showing engineers how to use the part of the system being tested and how that system is intended to work.

在實現防止bug之後，測試最重要的目的是提高工程師的生產效率。與範圍更廣的測試相比，單元測試有許多特性，使其成為優化生產效率的絕佳方式:

- 根據谷歌對測試規模的定義，它們往往是小型的。小型測試是快速和確定的，允許開發人員頻繁地執行它們，作為他們工作流程的一部分，並獲得即時反饋。
- 單元測試往往很容易與正在測試的程式碼同時編寫，允許工程師將他們的測試集中在他們正在工作的程式碼上，而不需要建立和理解一個更大的系統。
- 單元測試促進高水平的測試覆蓋率，因為它們快速且易於編寫。高測試覆蓋率使工程師能夠滿懷信心地進行更改，確保他們不會破壞任何東西。
- 由於每個單元測試在概念上都很簡單，並且都集中在系統的特定部分，因此，它們往往會使人們很容易理解失敗時的錯誤。
- 它們可以作為文件和例子，向工程師展示如何使用被測試的系統部分，以及該系統的預期工作方式。

Due to their many advantages, most tests written at Google are unit tests, and as a rule of thumb, we encourage engineers to aim for a mix of about 80% unit tests and 20% broader-scoped tests. This advice, coupled with the ease of writing unit tests and the speed with which they run, means that engineers run a *lot* of unit tests—it’s not at all unusual for an engineer to execute thousands of unit tests (directly or indirectly) during the average workday.

由於單元測試有很多優點，在谷歌寫的大多數測試都是單元測試，作為經驗法則，我們鼓勵工程師把80%的單元測試和20%的範圍更廣的測試混合起來。這個建議，再加上編寫單元測試的簡易性和執行速度，意味著工程師要執行*多個*單元測試——一個工程師在平均工作日中執行數千個單元測試（直接或間接）是很正常的。

Because they make up such a big part of engineers’ lives, Google puts a lot of focus on *test maintainability*. Maintainable tests are ones that “just work”: after writing them, engineers don’t need to think about them again until they fail, and those failures indicate real bugs with clear causes. The bulk of this chapter focuses on exploring the idea of maintainability and techniques for achieving it.

因為測試在工程師的生活中佔了很大一部分，所以谷歌非常重視*測試*的可維護性。可維護的測試是那些 "正常工作 "的測試：在寫完測試後，工程師不需要再考慮它們，直到它們失敗，而這些失敗表明有明確原因的真正錯誤。本章的主要內容是探討可維護性的概念和實現它的技術。

## The Importance of Maintainability  可維護性的重要性

Imagine this scenario: Mary wants to add a simple new feature to the product and is able to implement it quickly, perhaps requiring only a couple dozen lines of code. But when she goes to check in her change, she gets a screen full of errors back from the automated testing system. She spends the rest of the day going through those failures one by one. In each case, the change introduced no actual bug, but broke some of the assumptions that the test made about the internal structure of the code, requiring those tests to be updated. Often, she has difficulty figuring out what the tests were trying to do in the first place, and the hacks she adds to fix them make those tests even more difficult to understand in the future. Ultimately, what should have been a quick job ends up taking hours or even days of busywork, killing Mary’s productivity and sapping her morale.

想象一下這個場景：Mary希望向產品新增一個簡單的新功能，並且能夠快速實現它，可能只需要幾十行程式碼。但是，當她去檢查她的改動，她從自動測試系統那裡得到了滿屏的錯誤。她花了一天的時間來逐一檢查這些錯誤。在每種情況下，更改都沒有引入實際的bug，但打破了測試對程式碼內部結構的一些設定，需要更新這些測試。通常情況下，她很難弄清楚這些測試一開始要做什麼，而她為修復它們而新增的黑操作使得這些測試在以後更難理解。最終，本來應該是一份快速的工作，結果卻要花上幾個小時甚至幾天的時間忙碌，扼殺了Mary的工作效率，消磨了她的士氣。

Here, testing had the opposite of its intended effect by draining productivity rather than improving it while not meaningfully increasing the quality of the code under test. This scenario is far too common, and Google engineers struggle with it every day. There’s no magic bullet, but many engineers at Google have been working to develop sets of patterns and practices to alleviate these problems, which we encourage the rest of the company to follow.

在這裡，測試產生了與預期相反的效果，它消耗了生產力，而不是提高生產效率，同時沒有顯著提高被測試程式碼的品質。這種情況太普遍了，谷歌工程師每天都在與之鬥爭。沒有什麼靈丹妙藥，但谷歌的許多工程師一直在努力開發一套模式和實踐來緩解這些問題，我們鼓勵公司的其他人效仿。

The problems Mary ran into weren’t her fault, and there was nothing she could have done to avoid them: bad tests must be fixed before they are checked in, lest they impose a drag on future engineers. Broadly speaking, the issues she encountered fall into two categories. First, the tests she was working with were *brittle*: they broke in response to a harmless and unrelated change that introduced no real bugs. Second, the tests were *unclear*: after they were failing, it was difficult to determine what was wrong, how to fix it, and what those tests were supposed to be doing in the first place.

Mary遇到的問題不是她的錯，而且她也沒有辦法避免這些問題：糟糕的測試必須在出現之前被修復，以免它們給未來的工程師帶來阻力。概括地說，她遇到的問題分為兩類別。首先，她所使用的測試是很脆弱的：它們在應對一個無害的、不相關的變化時，沒有引入真正的bug而損壞。第二，測試不明確：在測試失敗後，很難確定哪裡出了問題，如何修復它，以及這些測試最初應該做什麼。

## Preventing Brittle Tests  預防脆性測試

As just defined, a brittle test is one that fails in the face of an unrelated change to production code that does not introduce any real bugs.[^1] Such tests must be diagnosed and fixed by engineers as part of their work. In small codebases with only a few engineers, having to tweak a few tests for every change might not be a big problem. But if a team regularly writes brittle tests, test maintenance will inevitably consume a larger and larger proportion of the team’s time as they are forced to comb through an increasing number of failures in an ever-growing test suite. If a set of tests needs to be manually tweaked by engineers for each change, calling it an “automated test suite” is a bit of a stretch!

正如剛才所定義的，脆性測試是指在面對不相關的程式程式碼變化時失敗的測試，這些變化不會引入任何真正的錯誤。在只有幾個工程師的小型程式碼函式庫中，每次修改都要調整一些測試，這可能不是一個大問題。但是，如果一個團隊經常寫脆弱測試，測試維護將不可避免地消耗團隊越來越多的時間，因為他們不得不在不斷增長的測試套件中梳理越來越多的失敗。如果一套測試需要工程師為每一個變化進行手動調整，稱其為 "自動化測試套件"就有點牽強了！

Brittle tests cause pain in codebases of any size, but they become particularly acute at Google’s scale. An individual engineer might easily run thousands of tests in a single day during the course of their work, and a single large-scale change (see [Chapter 22](#_bookmark1935)) can trigger hundreds of thousands of tests. At this scale, spurious breakages that affect even a small percentage of tests can waste huge amounts of engineering time. Teams at Google vary quite a bit in terms of how brittle their test suites are, but we’ve identified a few practices and patterns that tend to make tests more robust to change.

脆弱測試在任何規模的程式碼函式庫中都會造成痛苦，但在谷歌的規模中，它們變得尤為嚴重。一個單獨的工程師在工作過程中，可能在一天內就會輕易地執行數千個測試，而一個大規模的變化（見第22章）可能會引發數十萬個測試。在這種規模下，即使是影響一小部分測試的誤報故障也會浪費大量的工程時間。谷歌的團隊在測試套件的脆弱性方面存在很大差異，但我們已經確定了一些實踐和模式，這些實踐和模式傾向於使測試變得更健壯，更易於更改。

> [^1]: Note that this is slightly different from a flaky test, which fails nondeterministically without any change to production code./
> 1  注意，這與不穩定測試略有不同，不穩定測試是在不改變生產程式碼的情況下非確定性地失敗。

### Strive for Unchanging Tests  力求穩定的測試

Before talking about patterns for avoiding brittle tests, we need to answer a question: just how often should we expect to need to change a test after writing it? Any time spent updating old tests is time that can’t be spent on more valuable work. Therefore, *the ideal test is unchanging*: after it’s written, it never needs to change unless the requirements of the system under test change.

在討論避免脆性測試的模式之前，我們需要回答一個問題：編寫測試後，我們應該多久更改一次測試？任何花在更新舊測試上的時間都不能花在更有價值的工作上。因此，*理想的測試是不變的：*在編寫之後，它永遠不需要更改，除非被測系統的需求發生變化。

What does this look like in practice? We need to think about the kinds of changes that engineers make to production code and how we should expect tests to respond to those changes. Fundamentally, there are four kinds of changes:

- *Pure refactorings*  
	When an engineer refactors the internals of a system without modifying its interface, whether for performance, clarity, or any other reason, the system’s tests shouldn’t need to change. The role of tests in this case is to ensure that the refactoring didn’t change the system’s behavior. Tests that need to be changed during a refactoring indicate that either the change is affecting the system’s behavior and isn’t a pure refactoring, or that the tests were not written at an appropriate level of abstraction. Google’s reliance on large-scale changes (described in [Chapter 22](#_bookmark1935)) to do such refactorings makes this case particularly important for us.

- *New features*  
	When an engineer adds new features or behaviors to an existing system, the system’s existing behaviors should remain unaffected. The engineer must write new tests to cover the new behaviors, but they shouldn’t need to change any existing tests. As with refactorings, a change to existing tests when adding new features suggest unintended consequences of that feature or inappropriate tests.

- *Bug fixes*  
	Fixing a bug is much like adding a new feature: the presence of the bug suggests that a case was missing from the initial test suite, and the bug fix should include that missing test case. Again, bug fixes typically shouldn’t require updates to existing tests.

- *Behavior changes*  
	Changing a system’s existing behavior is the one case when we expect to have to make updates to the system’s existing tests. Note that such changes tend to be significantly more expensive than the other three types. A system’s users are likely to rely on its current behavior, and changes to that behavior require coordination with those users to avoid confusion or breakages. Changing a test in this case indicates that we’re breaking an explicit contract of the system, whereas changes in the previous cases indicate that we’re breaking an unintended contract. Low- level libraries will often invest significant effort in avoiding the need to ever make a behavior change so as not to break their users.

這在實踐中是什麼樣子的呢？我們需要考慮工程師對生產程式碼所做的各種修改，以及我們應該如何期望測試對這些修改做出反應。從根本上說，有四種更改：

- *純粹的重構*  
  當工程師在不修改系統介面的情況下重構系統內部時，無論是出於效能、清晰度還是任何其他原因，系統的測試都不需要更改。在這種情況下，測試的作用是確保重構沒有改變系統的行為。在重構過程中需要改變的測試表明，要麼變化影響了系統的行為，不是純粹的重構，要麼測試沒有寫在適當的抽象水平上。Google依靠大規模的變化（在第22章中描述）來做這樣的重構，使得這種情況對我們特別重要。

- *新功能*  
  當工程師向現有系統新增新的功能或行為時，系統的現有行為應該不受影響。工程師必須編寫新的測試來覆蓋新的行為，但他們不應該需要改變任何現有的測試。與重構一樣，在新增新功能時，對現有測試的改變表明該功能的非預期後果或不適當的測試。

- *Bug修復*  
  修復bug與新增新功能很相似：bug的存在表明初始測試套件中缺少一個案例，bug修復應該包括缺少的測試案例。同樣，錯誤修復通常不需要對現有的測試進行更新。

- *行為改變*  
  當我們期望必須對系統的現有測試進行更新時，更改系統的現有行為就是一種情況。請注意，這種變化往往比其他三種類型的測試代價要高得多。系統的使用者可能依賴於其當前行為，而對該行為的更改需要與這些使用者進行協調，以避免混淆或中斷。在這種情況下改變測試表明我們正在破壞系統的一個明確的契約，而在前面的情況下改變則表明我們正在破壞一個非預期的契約。基礎類別函式庫往往會投入大量的精力來避免需要進行行為的改變，以免破壞他們的使用者。

The takeaway is that after you write a test, you shouldn’t need to touch that test again as you refactor the system, fix bugs, or add new features. This understanding is what makes it possible to work with a system at scale: expanding it requires writing only a small number of new tests related to the change you’re making rather than potentially having to touch every test that has ever been written against the system. Only breaking changes in a system’s behavior should require going back to change its tests, and in such situations, the cost of updating those tests tends to be small relative to the cost of updating all of the system’s users.

啟示是，在編寫測試之後，在重構系統、修復bug或新增新功能時，不需要再次接觸該測試。這種理解使大規模使用系統成為可能：擴充套件系統只需要寫少量的與你所做的改變有關的新測試，而不是可能要觸動所有針對該系統寫過的測試。只有對系統行為的破壞性更改才需要返回以更改其測試，在這種情況下，更新這些測試的成本相對於更新所有系統使用者的成本往往很小。

### Test via Public APIs  透過公共API進行測試

Now that we understand our goal, let’s look at some practices for making sure that tests don’t need to change unless the requirements of the system being tested change. By far the most important way to ensure this is to write tests that invoke the system being tested in the same way its users would; that is, make calls against its public API [rather than its implementation details](https://oreil.ly/ijat0). If tests work the same way as the system’s users, by definition, change that breaks a test might also break a user. As an additional bonus, such tests can serve as useful examples and documentation for users.

現在我們瞭解了我們的目標，讓我們看看一些做法，以確保測試不需要改變，除非被測試系統的需求改變。到目前為止，確保這一點的最重要的方法是編寫測試，以與使用者相同的方式呼叫正在測試的系統；也就是說，針對其公共API[而不是其實現細節](https://oreil.ly/ijat0)進行呼叫。如果測試的工作方式與系統的使用者相同，根據定義，破壞測試的變化也可能破壞使用者。作為一個額外的好處，這樣的測試可以作為使用者的有用的例子和文件。

Consider [Example 12-1](#_bookmark959), which validates a transaction and saves it to a database.

考慮例12-1，它驗證了一個事務並將其儲存到資料庫中。

*Example* *12-1.* *A transaction API *  *實例12-1.事務API*

```java 
public void processTransaction(Transaction transaction) {
    if(isValid(transaction)) {
        saveToDatabase(transaction);
    }
}
private boolean isValid(Transaction t) {
    return t.getAmount() < t.getSender().getBalance();
}
private void saveToDatabase(Transaction t) {
    String s = t.getSender() + "," + t.getRecipient() + "," + t.getAmount();
    database.put(t.getId(), s);
}
public void setAccountBalance(String accountName, int balance) {
    // Write the balance to the database directly
}
public void getAccountBalance(String accountName) {
    // Read transactions from the database to determine the account balance
}
```

A tempting way to test this code would be to remove the “private” visibility modifiers and test the implementation logic directly, as demonstrated in Example 12-2.

測試這段程式碼的一個誘人的方法是去掉 "私有 "可見修飾符，直接測試實現邏輯，如例12-2所示。

*Example 12-2. A naive test of a transaction API’s implementation*  *例12-2.事務 API 實現的簡單測試*

```java
@Test
public void emptyAccountShouldNotBeValid() {
    assertThat(processor.isValid(newTransaction().setSender(EMPTY_ACCOUNT))).isFalse();
}

@Test
public void shouldSaveSerializedData() {
    processor.saveToDatabase(newTransaction().setId(123).setSender("me").setRecipient("you").setAmount(100));
    assertThat(database.get(123)).isEqualTo("me,you,100");
}

```

This test interacts with the transaction processor in a much different way than its real users would: it peers into the system’s internal state and calls methods that aren’t publicly exposed as part of the system’s API. As a result, the test is brittle, and almost any refactoring of the system under test (such as renaming its methods, factoring them out into a helper class, or changing the serialization format) would cause the test to break, even if such a change would be invisible to the class’s real users.

此測試與事務處理器的互動方式與其實際使用者的互動方式大不相同：它窺視系統的內部狀態並呼叫系統API中未公開的方法。因此，測試是脆弱的，幾乎任何對被測系統的重構（例如重新命名其方法、將其分解為輔助類別或更改序列化格式）都會導致測試中斷，即使此類別更改對類別的實際使用者是不可見的。

Instead, the same test coverage can be achieved by testing only against the class’s public API, as shown in Example 12-3.[^2]

相反，同樣的測試覆蓋率可以透過只測試類別的公共 API 來實現，如例 12-3.2 所示。

*Example 12-3. Testing the public API*  *例12-3. 測試公共API*

```java
@Test
public void shouldTransferFunds() {
    processor.setAccountBalance("me", 150);
    processor.setAccountBalance("you", 20);
    processor.processTransaction(newTransaction().setSender("me").setRecipient("you").setAmount(100));
    assertThat(processor.getAccountBalance("me")).isEqualTo(50);
    assertThat(processor.getAccountBalance("you")).isEqualTo(120);
}

@Test
public void shouldNotPerformInvalidTransactions() {
    processor.setAccountBalance("me", 50);
    processor.setAccountBalance("you", 20);
    processor.processTransaction(newTransaction().setSender("me").setRecipient("you").setAmount(100));
    assertThat(processor.getAccountBalance("me")).isEqualTo(50);
    assertThat(processor.getAccountBalance("you")).isEqualTo(20);
}

```

Tests using only public APIs are, by definition, accessing the system under test in the same manner that its users would. Such tests are more realistic and less brittle because they form explicit contracts: if such a test breaks, it implies that an existing user of the system will also be broken. Testing only these contracts means that you’re free to do whatever internal refactoring of the system you want without having to worry about making tedious changes to tests.

根據定義，僅使用公共API的測試是以與使用者相同的方式訪問被測系統。這樣的測試更現實，也不那麼脆弱，因為它們形成了明確的契約：如果這樣的測試失敗，它意味著系統的現有使用者也將失敗。只測試這些契約意味著你可以自由地對系統進行任何內部重構，而不必擔心對測試進行繁瑣的更改。

> [^2]:	This is sometimes called the "Use the front door first principle.”/
>
> 2   這有時被稱為“使用前門優先原則”

It’s not always clear what constitutes a “public API,” and the question really gets to the heart of what a “unit” is in unit testing. Units can be as small as an individual function or as broad as a set of several related packages/modules. When we say “public API” in this context, we’re really talking about the API exposed by that unit to third parties outside of the team that owns the code. This doesn’t always align with the notion of visibility provided by some programming languages; for example, classes in Java might define themselves as “public” to be accessible by other packages in the same unit but are not intended for use by other parties outside of the unit. Some languages like Python have no built-in notion of visibility (often relying on conventions like prefixing private method names with underscores), and build systems like Bazel can further restrict who is allowed to depend on APIs declared public by the programming language.

什麼是 "公共API "並不總是很清楚，這個問題實際上涉及到單元測試中的 "單元 "的核心。單元可以小到一個單獨的函式，也可以大到由幾個相關的包/模組組成的集合。當我們在這裡說 "公共API "時，我們實際上是在談論該單元暴露給擁有該程式碼的團隊之外的第三方的API。這並不總是與某些程式語言提供的可見性概念一致；例如，Java中的類別可能將自己定義為 "公共"，以便被同一單元中的其他包所訪問，但並不打算供該單元之外的其他方使用。有些語言，如Python，沒有內建的可見性概念（通常依靠慣例，如在私有方法名稱前加上下劃線），而像Bazel這樣的建構系統可以進一步限制誰可以依賴程式語言所宣告的公共API。

Defining an appropriate scope for a unit and hence what should be considered the public API is more art than science, but here are some rules of thumb:

- If a method or class exists only to support one or two other classes (i.e., it is a “helper class”), it probably shouldn’t be considered its own unit, and its functionality should be tested through those classes instead of directly.
- If a package or class is designed to be accessible by anyone without having to consult with its owners, it almost certainly constitutes a unit that should be tested directly, where its tests access the unit in the same way that the users would.
- If a package or class can be accessed only by the people who own it, but it is designed to provide a general piece of functionality useful in a range of contexts (i.e., it is a “support library”), it should also be considered a unit and tested directly. This will usually create some redundancy in testing given that the support library’s code will be covered both by its own tests and the tests of its users. However, such redundancy can be valuable: without it, a gap in test coverage could be introduced if one of the library’s users (and its tests) were ever removed.

為一個單元定義一個合適的範圍，因此應該將其視為公共API，這與其說是科學，不如說是藝術，但這裡有一些經驗法則：

- 如果一個方法或類別的存在只是為了支援一兩個其他的類別（即，它是一個 "輔助類別"），它可能不應該被認為是自己的單元，它的功能應該透過這些類別而不是直接測試。

- 如果一個包或類別被設計成任何人都可以訪問，而不需要諮詢其所有者，那麼它幾乎肯定構成了一個應該直接測試的單元，它的測試以使用者的方式訪問該單元。

- 如果一個包或類別只能由擁有它的人訪問，但它的設計目的是提供在各種上下文中有用的通用功能（即，它是一個“支援函式庫”），也應將其視為一個單元並直接進行測試。這通常會在測試中產生一些冗餘，因為支援函式庫的程式碼會被它自己的測試和使用者的測試所覆蓋。然而，這種冗餘可能是有價值的：如果沒有它，如果函式庫的一個使用者（和它的測試）被刪除，測試覆蓋率就會出現缺口。

At Google, we’ve found that engineers sometimes need to be persuaded that testing via public APIs is better than testing against implementation details. The reluctance is understandable because it’s often much easier to write tests focused on the piece of code you just wrote rather than figuring out how that code affects the system as a whole. Nevertheless, we have found it valuable to encourage such practices, as the extra upfront effort pays for itself many times over in reduced maintenance burden. Testing against public APIs won’t completely prevent brittleness, but it’s the most important thing you can do to ensure that your tests fail only in the event of meaningful changes to your system.

在谷歌，我們發現工程師有時需要被說服，透過公共API進行測試比針對實現細節進行測試要好。這種不情願的態度是可以理解的，因為寫測試的重點往往是你剛剛寫的那段程式碼，而不是弄清楚這段程式碼是如何影響整個系統的。然而，我們發現鼓勵這種做法是很有價值的，因為額外的前期努力在減少維護負擔方面得到了許多倍的回報。針對公共API的測試並不能完全防止脆性，但這是你能做的最重要的事情，以確保你的測試只在系統發生有意義的變化時才失敗。

### Test State, Not Interactions  測試狀態，而不是互動

Another way that tests commonly depend on implementation details involves not which methods of the system the test calls, but how the results of those calls are verified. In general, there are two ways to verify that a system under test behaves as expected. With *state testing*, you observe the system itself to see what it looks like after invoking with it. With *interaction testing*, you instead check that the system took an expected sequence of actions on its collaborators [in response to invoking it](https://oreil.ly/3S8AL). Many tests will perform a combination of state and interaction validation.

測試通常依賴於實現細節的另一種方法不是測試呼叫系統的哪些方法，而是如何驗證這些呼叫的結果。通常，有兩種方法可以驗證被測系統是否按預期執行。透過*狀態測試*，你觀察系統本身，看它在呼叫後是什麼樣子。透過*互動測試*，你要檢查系統是否對其合作者採取了預期的行動序列[以響應呼叫它](https://oreil.ly/3S8AL)。許多測試將執行狀態和互動驗證的組合。

Interaction tests tend to be more brittle than state tests for the same reason that it’s more brittle to test a private method than to test a public method: interaction tests check *how* a system arrived at its result, whereas usually you should care only *what* the result is. [Example 12-4 ](#_bookmark971)illustrates a test that uses a test double (explained further in [Chapter 13](#_bookmark1056)) to verify how a system interacts with a database.

互動測試往往比狀態測試更脆弱，原因與測試一個私有方法比測試一個公共方法更脆的原因相同：互動測試檢查系統是*如何*得到結果的，而通常你只應該關心結果是*什麼*。例12-4展示了一個測試，它使用一個測試替換（在第13章中進一步解釋）來驗證一個系統如何與資料庫互動。

*Example  12-4. A brittle interaction test*  *例12-4.  脆弱性相互作用測試*

```java
@Test
public void shouldWriteToDatabase() {
    accounts.createUser("foobar");
    verify(database).put("foobar");
}
```

The test verifies that a specific call was made against a database API, but there are a couple different ways it could go wrong:
- If a bug in the system under test causes the record to be deleted from the database shortly after it was written, the test will pass even though we would have wanted it to fail.

- If the system under test is refactored to call a slightly different API to write an equivalent record, the test will fail even though we would have wanted it to pass.

該測試驗證了對資料庫API的特定呼叫，但有兩種不同的方法可能會出錯：

- 如果被測系統中的錯誤導致記錄在寫入後不久從資料庫中刪除，那麼即使我們希望它失敗，測試也會透過。
- 如果對被測系統進行重構，以呼叫稍有不同的API來編寫等效記錄，那麼即使我們希望測試透過，測試也會失敗。

It’s much less brittle to directly test against the state of the system, as demonstrated in Example 12-5.

如例12-5所示，直接對系統的狀態進行測試就不那麼脆弱了。

*Example 12-5. Testing against state*  *例12-5. 針對狀態的測試*

```java
@Test
public void shouldCreateUsers() {
    accounts.createUser("foobar");
    assertThat(accounts.getUser("foobar")).isNotNull();
}
```

This test more accurately expresses what we care about: the state of the system under test after interacting with it.

這種測試更準確地表達了我們所關心的：被測系統與之互動後的狀態。

The most common reason for problematic interaction tests is an over reliance on mocking frameworks. These frameworks make it easy to create test doubles that record and verify every call made against them, and to use those doubles in place of real objects in tests. This strategy leads directly to brittle interaction tests, and so we tend to prefer the use of real objects in favor of mocked objects, as long as the real objects are fast and deterministic.

互動測試出現問題的最常見原因是過度依賴mocking框架。這些框架可以很容易地建立測試替換，記錄並驗證針對它們的每個呼叫，並在測試中使用這些替換來代替真實物件。這種策略直接導致了脆弱的互動測試，因此我們傾向於使用真實物件而不是模擬物件，只要真實物件是快速和確定的。

## Writing Clear Tests  編寫清晰的測試

Sooner or later, even if we’ve completely avoided brittleness, our tests will fail. Failure is a good thing—test failures provide useful signals to engineers, and are one of the main ways that a unit test provides value.
Test failures happen for one of two reasons:[^3]

- The system under test has a problem or is incomplete. This result is exactly what tests are designed for: alerting you to bugs so that you can fix them.
- The test itself is flawed. In this case, nothing is wrong with the system under test, but the test was specified incorrectly. If this was an existing test rather than one that you just wrote, this means that the test is brittle. The previous section discussed how to avoid brittle tests, but it’s rarely possible to eliminate them entirely.

總有一天，即使我們已經完全避免了脆弱性，我們的測試也會失敗。失敗是一件好事——測試失敗為工程師提供了有用的訊號，也是單元測試提供價值的主要方式之一。
測試失敗有兩個原因：

- 被測系統有問題或不完整。這個結果正是測試的設計目的：提醒你注意bug，以便你能修復它們。
- 測試本身是有缺陷的。在這種情況下，被測系統沒有任何問題，但測試的指定是不正確的。如果這是一個現有的測試，而不是你剛寫的測試，這意味著測試是脆弱的。上一節討論瞭如何避免脆性測試，但很少有可能完全消除它們。

When a test fails, an engineer’s first job is to identify which of these cases the failure falls into and then to diagnose the actual problem. The speed at which the engineer can do so depends on the test’s clarity. A clear test is one whose purpose for existing and reason for failing is immediately clear to the engineer diagnosing a failure. Tests fail to achieve clarity when their reasons for failure aren’t obvious or when it’s difficult to figure out why they were originally written. Clear tests also bring other benefits, such as documenting the system under test and more easily serving as a basis for new tests.

當測試失敗時，工程師的首要工作是確定失敗屬於哪種情況，然後診斷出實際問題。工程師定位問題的速度取決於測試的清晰程度。清晰的測試是指工程師在診斷故障時，立即明確其存在目的和故障原因的測試。如果測試失敗的原因不明顯，或者很難弄清楚最初寫這些測試的原因，那麼測試就無法達到清晰的效果。清晰的測試還能帶來其他的好處，比如記錄被測系統，更容易作為新測試的基礎。

Test clarity becomes significant over time. Tests will often outlast the engineers who wrote them, and the requirements and understanding of a system will shift subtly as it ages. It’s entirely possible that a failing test might have been written years ago by an engineer no longer on the team, leaving no way to figure out its purpose or how to fix it. This stands in contrast with unclear production code, whose purpose you can usually determine with enough effort by looking at what calls it and what breaks when it’s removed. With an unclear test, you might never understand its purpose, since removing the test will have no effect other than (potentially) introducing a subtle hole in test coverage.

隨著時間的推移，測試的清晰度變得非常重要。測試往往比編寫測試的工程師的時間更長，而且隨著時間的推移，對系統的要求和理解會發生微妙的變化。一個失敗的測試完全有可能是多年前由一個已經不在團隊中的工程師寫的，沒有辦法弄清楚其目的或如何修復它。這與不明確的生產程式碼形成了鮮明的對比，你通常可以透過檢視呼叫程式碼的內容和刪除程式碼後的故障來確定其目的。對於一個不明確的測試，你可能永遠不會明白它的目的，因為刪除該測試除了（潛在地）在測試覆蓋率中引入一個細微的漏洞之外沒有任何影響。

In the worst case, these obscure tests just end up getting deleted when engineers can’t figure out how to fix them. Not only does removing such tests introduce a hole in test coverage, but it also indicates that the test has been providing zero value for perhaps the entire period it has existed (which could have been years).

在最壞的情況下，這些晦澀難懂的測試最終會被刪除，因為工程師不知道如何修復它們。刪除這些測試不僅會在測試覆蓋率上帶來漏洞，而且還表明該測試在其存在的整個期間（可能是多年）一直提供零價值。

For a test suite to scale and be useful over time, it’s important that each individual test in that suite be as clear as possible. This section explores techniques and ways of thinking about tests to achieve clarity.

為了使測試套件能夠隨時間擴充套件並變得有用，套件中的每個測試都儘可能清晰是很重要的。本節探討了為實現清晰性而考慮測試的技術和方法。

> [^3]: These are also the same two reasons that a test can be “flaky.” Either the system under test has a nondeterministic fault, or the test is flawed such that it sometimes fails when it should pass./
> 3   這也是測試可能“不穩定”的兩個原因。要麼被測系統存在不確定性故障，要麼測試存在缺陷，以至於在透過測試時有時會失敗。

### Make Your Tests Complete and Concise  確保你的測試完整和簡明

Two high-level properties that help tests achieve clarity are completeness and conciseness. A test is complete when its body contains all of the information a reader needs in order to understand how it arrives at its result. A test is concise when it contains no other distracting or irrelevant information. Example 12-6 shows a test that is neither complete nor concise:

幫助測試實現清晰的兩個高階屬性是完整性和簡潔性。一個測試是完整的，當它的主體包含讀者需要的所有資訊，以瞭解它是如何得出結果的。當一個測試不包含其他分散注意力的或不相關的資訊時，它就是簡潔的。例12-6顯示了一個既不完整也不簡潔的測試：

*Example 12-6. An incomplete and cluttered test*   *例12-6. 一個不完整且雜亂的測試*

```java
@Test
public void shouldPerformAddition() {
    Calculator calculator = new Calculator(new RoundingStrategy(), "unused", ENABLE_COSINE_FEATURE, 0.01, calculusEngine, false);
    int result = calculator.calculate(newTestCalculation());
    assertThat(result).isEqualTo(5); // Where did this number come from?
}
```

The test is passing a lot of irrelevant information into the constructor, and the actual important parts of the test are hidden inside of a helper method. The test can be made more complete by clarifying the inputs of the helper method, and more concise by using another helper to hide the irrelevant details of constructing the calculator, as illustrated in Example 12-7.

測試將大量不相關的資訊傳遞給建構函式，測試的實際重要部分隱藏在輔助方法中。透過澄清輔助方法的輸入，可以使測試更加完整，透過使用另一個輔助隱藏建構計算器的無關細節，可以使測試更加簡潔，如示例12-7所示。

*Example 12-7. A complete, concise test*  *實例 12-7. A. 完整且簡潔的測驗*

```java
@Test
public void shouldPerformAddition() { 
    Calculator calculator = newCalculator();
    int result = calculator.calculate(newCalculation(2, Operation.PLUS, 3));
    assertThat(result).isEqualTo(5);
}
```

Ideas we discuss later, especially around code sharing, will tie back to completeness and conciseness. In particular, it can often be worth violating the DRY (Don’t Repeat Yourself) principle if it leads to clearer tests. Remember: a test’s body should contain all of the information needed to understand it without containing any irrelevant or distracting information.

我們稍後討論的觀點，特別是圍繞程式碼共享，將與完整性和簡潔性掛鉤。需要注意的是，如果能使測試更清晰，違反DRY（不要重複自己）原則通常是值得的。記住：一個測試的主體應該包含理解它所需要的所有資訊，而不包含任何無關或分散的資訊。

### Test Behaviors, Not Methods  測試行為，而不是方法

The first instinct of many engineers is to try to match the structure of their tests to the structure of their code such that every production method has a corresponding test method. This pattern can be convenient at first, but over time it leads to problems: as the method being tested grows more complex, its test also grows in complexity and becomes more difficult to reason about. For example, consider the snippet of code in Example 12-8, which displays the results of a transaction.

許多工程師的第一直覺是試圖將他們的測試結構與他們的程式碼結構相匹配，這樣每個產品方法都有一個相應的測試方法。這種模式一開始很方便，但隨著時間的推移，它會導致問題：隨著被測試的方法越來越複雜，它的測試也越來越複雜，變得越來越難以理解。例如，考慮例12-8中的程式碼片段，它顯示了一個事務的結果。

*Example 12-8. A transaction snippet*  *例12-8. 一個事務片段*

```java
public void displayTransactionResults(User user, Transaction transaction) {
    ui.showMessage("You bought a " + transaction.getItemName());
    if(user.getBalance() < LOW_BALANCE_THRESHOLD) {
        ui.showMessage("Warning: your balance is low!");
    }
}
```
It wouldn’t be uncommon to find a test covering both of the messages that might be shown by the method, as presented in Example 12-9.

如例12-9所示，一個測試涵蓋了該方法可能顯示的兩個資訊，這並不罕見。

*Example 12-9. A method-driven test*   *例12-9. 方法驅動的測試*

```java
@Test
public void testDisplayTransactionResults() {
transactionProcessor.displayTransactionResults(newUserWithBalance(LOW_BALANCE_THRESHOLD.plus(dollars(2))), new Transaction("Some Item", dollars(3)));
    assertThat(ui.getText()).contains("You bought a Some Item");
    assertThat(ui.getText()).contains("your balance is low");
}

```

With such tests, it’s likely that the test started out covering only the first method. Later, an engineer expanded the test when the second message was added (violating the idea of unchanging tests that we discussed earlier). This modification sets a bad precedent: as the method under test becomes more complex and implements more functionality, its unit test will become increasingly convoluted and grow more and more difficult to work with.

對於這樣的測試，很可能一開始測試只包括第一個方法。後來，當第二條資訊被新增進來時，工程師擴充套件了測試（違反了我們前面討論的穩定的測試理念）。這種修改開創了一個不好的先例：隨著被測方法變得越來越複雜，實現的功能越來越多，其單元測試也會變得越來越複雜，越來越難以使用。

The problem is that framing tests around methods can naturally encourage unclear tests because a single method often does a few different things under the hood and might have several tricky edge and corner cases. There’s a better way: rather than writing a test for each method, write a test for each behavior.[^4] A behavior is any guarantee that a system makes about how it will respond to a series of inputs while in a particular state.[^5] Behaviors can often be expressed using the words “given,” “when,” and “then”: “Given that a bank account is empty, when attempting to withdraw money from it, then the transaction is rejected.” The mapping between methods and behaviors is many-to-many: most nontrivial methods implement multiple behaviors, and some behaviors rely on the interaction of multiple methods. The previous example can be rewritten using behavior-driven tests, as presented in Example 12-10.

問題是，圍繞方法測試框架自然會鼓勵不清晰測試，因為單個方法經常在背後下做一些不同的事情，可能有幾個棘手的邊緣和角落的情況。有一個更好的方法：與其為每個方法寫一個測試，不如為每個行為寫一個測試。 行為是一個系統對它在特定狀態下如何響應一系列輸入的任何保證。"鑑於一個銀行賬戶是空的，當試圖從該賬戶中取錢時，該交易被拒絕。" 方法和行為之間的對映是多對多的：大多數不重要的方法實現了多個行為，一些行為依賴於多個方法的互動。前面的例子可以用行為驅動的測試來重寫，如例12-10所介紹。

*Example 12-10. A behavior-driven test*   *例12-10. 行為驅動的測試*

```java
@Test
public void displayTransactionResults_showsItemName() {
    transactionProcessor.displayTransactionResults(new User(), new Transaction("Some Item"));
    assertThat(ui.getText()).contains("You bought a Some Item");
}

@Test
public void displayTransactionResults_showsLowBalanceWarning() {
    transactionProcessor.displayTransactionResults(newUserWithBalance(LOW_BALANCE_THRESHOLD.plus(dollars(2))), new Transaction("Some Item", dollars(3)));
    assertThat(ui.getText()).contains("your balance is low");
}

```

The extra boilerplate required to split apart the single test is more than worth it, and the resulting tests are much clearer than the original test. Behavior-driven tests tend to be clearer than method-oriented tests for several reasons. First, they read more like natural language, allowing them to be naturally understood rather than requiring laborious mental parsing. Second, they more clearly express cause and effect because each test is more limited in scope. Finally, the fact that each test is short and descriptive makes it easier to see what functionality is already tested and encourages engineers to add new streamlined test methods instead of piling onto existing methods.

拆分單個測試所需的額外範本檔案非常值得，並且最終的測試比原來測試更清晰。行為驅動測試往往比面向方法的測試更清晰，原因有幾個。首先，它們閱讀起來更像自然語言，讓人們自然地理解它們，而不需要語言繁瑣的心理分析。其次，它們更清楚地表達了因果關係，因為每個測試的範圍都更有限。最後，每個測試都很短且描述性強，這一事實使我們更容易看到已經測試了哪些功能，並鼓勵工程師新增新的簡潔測試方法，而不是堆積在現有方法上。

> 4	See https://testing.googleblog.com/2014/04/testing-on-toilet-test-behaviors-not.html and https://dannorth.net/introducing-bdd./
> 4 見 https://testing.googleblog.com/2014/04/testing-on-toilet-test-behaviors-not.html 和 https://dannorth.net/introducing-bdd。
>
> [^5]: Furthermore, a feature (in the product sense of the word) can be expressed as a collection of behaviors./
> 5 此外，一個特徵（在這個詞的產品意義上）可以被表達為一個行為的集合。

#### Structure tests to emphasize behaviors  強調行為的結構測試

Thinking about tests as being coupled to behaviors instead of methods significantly affects how they should be structured. Remember that every behavior has three parts: a “given” component that defines how the system is set up, a “when” component that defines the action to be taken on the system, and a “then” component that validates the result.[^6] Tests are clearest when this structure is explicit. Some frameworks like Cucumber and Spock directly bake in given/when/then. Other languages can use whitespace and optional comments to make the structure stand out, such as that shown in Example 12-11.

將測試視為與行為而非方法相耦合會顯著影響測試的結構。請記住，每個行為都有三個部分：一個是定義系統如何設定的 "given "元件，一個是定義對系統採取的行動的 "when "元件，以及一個驗證結果的 "then "元件。當此結構是顯式的時，測試是最清晰的。一些框架（如Cucumber和Spock）直接加入了given/when/then的功能支援。其他語言可以使用空格和可選註釋使結構突出，如示例12-11所示。

*Example 12-11. A well-structured test*  *例12-11. 一個結構良好的測試*

```java  
@Test
public void transferFundsShouldMoveMoneyBetweenAccounts() {
    // Given two accounts with initial balances of $150 and $20
    Account account1 = newAccountWithBalance(usd(150));
    Account account2 = newAccountWithBalance(usd(20));
    // When transferring $100 from the first to the second account
    bank.transferFunds(account1, account2, usd(100));
    // Then the new account balances should reflect the transfer 
    assertThat(account1.getBalance()).isEqualTo(usd(50));
    assertThat(account2.getBalance()).isEqualTo(usd(120));
}

```

This level of description isn’t always necessary in trivial tests, and it’s usually sufficient to omit the comments and rely on whitespace to make the sections clear. However, explicit comments can make more sophisticated tests easier to understand. This pattern makes it possible to read tests at three levels of granularity:

1. A reader can start by looking at the test method name (discussed below) to get a rough description of the behavior being tested.

2. If that’s not enough, the reader can look at the given/when/then comments for a formal description of the behavior.

3. Finally, a reader can look at the actual code to see precisely how that behavior is expressed.

這種程度的描述在瑣碎的測試中並不總是必要的，通常省略註釋並依靠空白來使各部分清晰。然而，明確的註釋可以使更復雜的測試更容易理解。這種模式使我們有可能在三個層次的粒度上閱讀測試。

1. 讀者可以從測試方法的名稱開始（下面討論），以獲得對被測試行為的粗略描述。

2.	如果這還不夠，讀者可以檢視given/when/then註釋，以獲得行為的正式描述。

3. 最後，讀者可以檢視實際程式碼，以準確地看到該行為是如何表達的。

This pattern is most commonly violated by interspersing assertions among multiple calls to the system under test (i.e., combining the “when” and “then” blocks). Merging the “then” and “when” blocks in this way can make the test less clear because it makes it difficult to distinguish the action being performed from the expected result.

最常見的違反模式是在對被測系統的多個呼叫之間穿插斷言（即，組合“when”和“then”塊）。以這種方式合併 "then "和 "when "塊會使測試不那麼清晰，因為它使人們難以區分正在執行的操作和預期結果。

When a test does want to validate each step in a multistep process, it’s acceptable to define alternating sequences of when/then blocks. Long blocks can also be made more descriptive by splitting them up with the word “and.” Example 12-12 shows what a relatively complex, behavior-driven test might look like.

當一個測試確實想驗證一個多步驟過程中的每個步驟時，定義when/then塊的交替序列是可以接受的。長的區塊也可以用 "and"字來分割，使其更具描述性。例12-12顯示了一個相對複雜的、行為驅動的測試是什麼樣子的。

*Example 12-12. Alternating when/then blocks within a test*   *例12-12. 在一個測試中交替使用when/then塊*

```java
@Test
public void shouldTimeOutConnections() {
    // Given two users
    User user1 = newUser();
    User user2 = newUser();
    // And an empty connection pool with a 10-minute timeout
    Pool pool = newPool(Duration.minutes(10));
    // When connecting both users to the pool
    pool.connect(user1);
    pool.connect(user2);
    // Then the pool should have two connections
    assertThat(pool.getConnections()).hasSize(2);
    // When waiting for 20 minutes
    clock.advance(Duration.minutes(20));
    // Then the pool should have no connections
    assertThat(pool.getConnections()).isEmpty();
    // And each user should be disconnected 
    assertThat(user1.isConnected()).isFalse();
    assertThat(user2.isConnected()).isFalse();
}

```

When writing such tests, be careful to ensure that you’re not inadvertently testing multiple behaviors at the same time. Each test should cover only a single behavior, and the vast majority of unit tests require only one “when” and one “then” block.

在編寫這種測試時，要注意確保你不會無意中同時測試多個行為。每個測試應該只覆蓋一個行為，絕大多數的單元測試只需要一個 "when "和一個 "then "塊。

> [^6]: These components are sometimes referred to as “arrange,” “act,” and “assert.”/
> 6 這些組成部分有時被稱為 "安排"、"行動 "和 "斷言"。

#### Name tests after the behavior being tested  以被測試的行為命名測試

Method-oriented tests are usually named after the method being tested (e.g., a test for the updateBalance method is usually called testUpdateBalance). With more focused behavior-driven tests, we have a lot more flexibility and the chance to convey useful information in the test’s name. The test name is very important: it will often be the first or only token visible in failure reports, so it’s your best opportunity to communicate the problem when the test breaks. It’s also the most straightforward way to express the intent of the test.

面向方法的測試通常以被測試的方法命名（例如，對 updateBalance 方法的測試通常稱為 testUpdateBalance）。對於更加集中的行為驅動的測試，我們有更多的靈活性，並有機會在測試的名稱中傳達有用的資訊。測試名稱非常重要：它通常是失敗報告中第一個或唯一一個可見的標記，所以當測試中斷時，它是你溝通問題的最好機會。它也是表達測試意圖的最直接的方式。

A test’s name should summarize the behavior it is testing. A good name describes both the actions that are being taken on a system and the expected outcome. Test names will sometimes include additional information like the state of the system or its environment before taking action on it. Some languages and frameworks make this easier than others by allowing tests to be nested within one another and named using strings, such as in Example 12-13, which uses Jasmine.

測試的名字應該概括它所測試的行為。一個好的名字既能描述在系統上採取的行動，又能描述預期的結果。測試名稱有時會包括額外的資訊，如系統或其環境的狀態。一些語言和框架允許測試相互巢狀，並使用字串命名，例如例12-13，其中使用了Jasmine，這樣做比其他語言和框架更容易。

*Example 12-13. Some sample nested naming patterns*  *例12-1. 一些巢狀命名模式的例子*

```java
describe("multiplication", function() {
    describe("with a positive number", function() {
        var positiveNumber = 10;
        it("is positive with another positive number", function() {
            expect(positiveNumber * 10).toBeGreaterThan(0);
        });
        it("is negative with a negative number", function() {
            expect(positiveNumber * -10).toBeLessThan(0);
        });
    });
    describe("with a negative number", function() {
        var negativeNumber = 10;
        it("is negative with a positive number", function() {
            expect(negativeNumber * 10).toBeLessThan(0);
        });
        it("is positive with another negative number", function() {
            expect(negativeNumber * -10).toBeGreaterThan(0);
        });
    });
});

```

Other languages require us to encode all of this information in a method name, leading to method naming patterns like that shown in Example 12-14.

其他語言要求我們在方法名中編碼所有這些資訊，導致方法的命名模式如例12-14所示。

*Example 12-14. Some sample method naming patterns*  例12-14. 一些示例方法的命名模式

```java
multiplyingTwoPositiveNumbersShouldReturnAPositiveNumber 
multiply_postiveAndNegative_returnsNegative 
divide_byZero_throwsException
````

Names like this are much more verbose than we’d normally want to write for methods in production code, but the use case is different: we never need to write code that calls these, and their names frequently need to be read by humans in reports. Hence, the extra verbosity is warranted.

像這樣的名字比我們通常為產品程式碼中的方法所寫的要囉嗦得多，但使用情況不同：我們從來不需要寫程式碼來呼叫這些方法，而且它們的名字經常需要由人類在報告中閱讀。因此，額外的言辭是有必要的。

Many different naming strategies are acceptable so long as they’re used consistently within a single test class. A good trick if you’re stuck is to try starting the test name with the word “should.” When taken with the name of the class being tested, this naming scheme allows the test name to be read as a sentence. For example, a test of a BankAccount class named shouldNotAllowWithdrawalsWhenBalanceIsEmpty can be read as “BankAccount should not allow withdrawals when balance is empty.” By reading the names of all the test methods in a suite, you should get a good sense of the behaviors implemented by the system under test. Such names also help ensure that the test stays focused on a single behavior: if you need to use the word “and” in a test name, there’s a good chance that you’re actually testing multiple behaviors and should be writing multiple tests!

許多不同的命名策略是可以接受的，只要它們在一個測試類別中使用一致。如果你陷入困境，一個好的技巧是嘗試用 "應該 "這個詞來開始測試名稱。當與被測類別的名稱一起使用時，這種命名方案允許將測試名稱作為一個句子來閱讀。例如，一個名為shouldNotAllowWithdrawalsWhenBalanceIsEmpty的BankAccount類別的測試可以被理解為 "BankAccount不應該允許在餘額為空時提款"。透過閱讀套件中所有測試方法的名稱，你應該對被測系統實現的行為有一個很好的瞭解。這樣的名字也有助於確保測試集中在單個行為上：如果你需要在測試名稱中使用 "and"這個詞，很有可能你實際上是在測試多個行為，應該寫多個測試!

### Don’t Put Logic in Tests  不要把邏輯放進測試中

Clear tests are trivially correct upon inspection; that is, it is obvious that a test is doing the correct thing just from glancing at it. This is possible in test code because each test needs to handle only a particular set of inputs, whereas production code must be generalized to handle any input. For production code, we’re able to write tests that ensure complex logic is correct. But test code doesn’t have that luxury—if you feel like you need to write a test to verify your test, something has gone wrong!

清晰的測試在檢查時通常是正確的；也就是說，很明顯，只要看一眼，測試就做了正確的事情。這在測試程式碼中是可能的，因為每個測試只需要處理一組特定的輸入，而產品程式碼必須被泛化以處理任何輸入。對於產品程式碼，我們能夠編寫測試，確保複雜的邏輯是正確的。但測試程式碼沒有那麼奢侈——如果你覺得你需要寫一個測試來驗證你的測試，那就說明出了問題！這是不可能的。

Complexity is most often introduced in the form of logic. Logic is defined via the imperative parts of programming languages such as operators, loops, and conditionals. When a piece of code contains logic, you need to do a bit of mental computation to determine its result instead of just reading it off of the screen. It doesn’t take much logic to make a test more difficult to reason about. For example, does the test in Example 12-15 look correct to you?

複雜性最常以邏輯的形式引入。邏輯是透過程式語言的指令部分來定義的，如運算子、迴圈和條件。當一段程式碼包含邏輯時，你需要做一些心理預期來確定其結果，而不是僅僅從螢幕上讀出來。不需要太多的邏輯就可以使一個測試變得更難理解。例如，例12-15中的測試在你看來是否正確？

*Example 12-15. Logic concealing a bug*  *例12-15. 掩蓋bug的邏輯*

```java
@Test
public void shouldNavigateToAlbumsPage() {
    String baseUrl = "http://photos.google.com/";
    Navigator nav = new Navigator(baseUrl);
    nav.goToAlbumPage();
    assertThat(nav.getCurrentUrl()).isEqualTo(baseUrl + "/albums");
}
```

There’s not much logic here: really just one string concatenation. But if we simplify the test by removing that one bit of logic, a bug immediately becomes clear, as demonstrated in Example 12-16.

這裡沒有什麼邏輯：實際上只是一個字串連線。但是，如果我們透過刪除這一點邏輯來簡化測試，一個錯誤就會立即變得清晰，如例12-16所示。

*Example 12-16. A test without logic reveals the bug*  *例12-16. 沒有邏輯的測試揭示了bug*

```java
@Test
public void shouldNavigateToPhotosPage() {
    Navigator nav = new Navigator("http://photos.google.com/");
    nav.goToPhotosPage();
    assertThat(nav.getCurrentUrl())).isEqualTo("http://photos.google.com//albums"); // Oops!
}
```

When the whole string is written out, we can see right away that we’re expecting two slashes in the URL instead of just one. If the production code made a similar mistake, this test would fail to detect a bug. Duplicating the base URL was a small price to pay for making the test more descriptive and meaningful (see the discussion of DAMP versus DRY tests later in this chapter).

當寫出整個字串時，我們可以立即看到，我們期望URL中有兩個斜槓，而不是一個。如果產品程式碼犯了類似的錯誤，此測試將無法檢測到錯誤。重複基本URL是為了使測試更具描述性和意義而付出的小代價（見本章後面關於DAMP與DRY測試的討論）。

If humans are bad at spotting bugs from string concatenation, we’re even worse at spotting bugs that come from more sophisticated programming constructs like loops and conditionals. The lesson is clear: in test code, stick to straight-line code over clever logic, and consider tolerating some duplication when it makes the test more descriptive and meaningful. We’ll discuss ideas around duplication and code sharing later in this chapter.

如果人類不善於發現來自字串連線的錯誤，那麼我們更不善於發現來自更復雜的程式設計結構的錯誤，如迴圈和條件。這個教訓很清晰：在測試程式碼中，堅持使用直線程式碼而不是複雜的邏輯，並在測試更具描述性的時候考慮容忍一些重複。我們將在本章後面討論關於重複和程式碼共享的想法。

### Write Clear Failure Messages  給出清晰的失敗資訊

One last aspect of clarity has to do not with how a test is written, but with what an engineer sees when it fails. In an ideal world, an engineer could diagnose a problem just from reading its failure message in a log or report without ever having to look at the test itself. A good failure message contains much the same information as the test’s name: it should clearly express the desired outcome, the actual outcome, and any relevant parameters.

清晰度的最後一個方面與測試的編寫方式無關，而是與工程師在測試失敗時看到的資訊有關。在一個理想的世界裡，工程師可以透過閱讀日誌或報告中的失敗資訊來診斷一個問題，而不需要看測試本身。一個好的故障資訊包含與測試名稱相同的資訊：它應該清楚地表達預期結果、實際結果和任何相關的引數。

Here’s an example of a bad failure message:

下面是一個糟糕失敗訊息的示例：

```Java
Test failed: account is closed
```

Did the test fail because the account was closed, or was the account expected to be closed and the test failed because it wasn’t? A better failure message clearly distinguishes the expected from the actual state and gives more context about the result:

測試失敗是因為帳戶已關閉，還是因為帳戶預期將關閉，而測試失敗是因為帳戶未關閉？一條更好的失敗訊息清楚地將預期狀態與實際狀態區分開來，並提供有關結果的更多上下文：

```java
Expected an account in state CLOSED, but got account:
<{name: "my-account", state: "OPEN"}
```

Good libraries can help make it easier to write useful failure messages. Consider the assertions in Example 12-17 in a Java test, the first of which uses classical JUnit asserts, and the second of which uses Truth, an assertion library developed by Google:

好的函式庫可以幫助我們更容易寫出有用的失敗資訊。考慮一下例12-17中Java測試中的斷言，第一個斷言使用了經典的JUnit斷言，第二個斷言使用了Truth，一個由Google開發的斷言函式庫：

*Example 12-17. An assertion using the Truth library*   *例12-17.  使用Truth函式庫的斷言*

```java
Set<String> colors = ImmutableSet.of("red", "green", "blue"); 
assertTrue(colors.contains("orange")); // JUnit 
assertThat(colors).contains("orange"); // Truth
```

Because the first assertion only receives a Boolean value, it is only able to give a generic error message like “expected `true` but was `false`,” which isn’t very informative in a failing test output. Because the second assertion explicitly receives the subject of the assertion, it is able to give a much more useful error message: AssertionError: <[red, green, blue]> should have contained `orange`.”

因為第一個斷言只接收一個布林值，所以它只能給出一個通用的錯誤資訊，如 "預期`true`，但得到的是`false`"，這在失敗的測試輸出中不是很有意義。因為第二個斷言明確地接收斷言的主題，它能夠給出一個更有用的錯誤資訊。AssertionError: <[red, green, blue]>應該包含`orange`"。

Not all languages have such helpers available, but it should always be possible to manually specify the important information in the failure message. For example, test assertions in Go conventionally look like Example 12-18.

並非所有的語言都有這樣的輔助工具，但總是可以手動指定失敗資訊中的重要資訊。例如，Go中的測試斷言通常看起來像例12-18。

*Example 12-18. A test assertion in Go*   *例12-18. Go中的測試斷言*

```golang
result: = Add(2, 3)
if result != 5 {
    t.Errorf("Add(2, 3) = %v, want %v", result, 5)
}
```

## Tests and Code Sharing: DAMP, Not DRY  測試和程式碼共享：DAMP，而不是DRY

One final aspect of writing clear tests and avoiding brittleness has to do with code sharing. Most software attempts to achieve a principle called DRY—“Don’t Repeat Yourself.” DRY states that software is easier to maintain if every concept is canonically represented in one place and code duplication is kept to a minimum. This approach is especially valuable in making changes easier because an engineer needs to update only one piece of code rather than tracking down multiple references. The downside to such consolidation is that it can make code unclear, requiring readers to follow chains of references to understand what the code is doing.

編寫清晰的測試和避免脆弱性的最後一個方面與程式碼共享有關。大多數軟體都試圖實現一個稱為DRY的原則——“不要重複你自己。”DRY指出，如果每個概念都在一個地方被規範地表示，並且程式碼重複保持在最低限度，那麼軟體就更容易維護。這種方法在簡化更改方面尤其有用，因為工程師只需要更新一段程式碼，而不需要追蹤多個參考。。這種合併的缺點是，它可能會使程式碼變得不清楚，需要讀者跟隨參考鏈來理解程式碼在做什麼。

In normal production code, that downside is usually a small price to pay for making code easier to change and work with. But this cost/benefit analysis plays out a little differently in the context of test code. Good tests are designed to be stable, and in fact you usually want them to break when the system being tested changes. So DRY doesn’t have quite as much benefit when it comes to test code. At the same time, the costs of complexity are greater for tests: production code has the benefit of a test suite to ensure that it keeps working as it becomes complex, whereas tests must stand by themselves, risking bugs if they aren’t self-evidently correct. As mentioned earlier, something has gone wrong if tests start becoming complex enough that it feels like they need their own tests to ensure that they’re working properly.

在通常的產品程式碼中，為了使程式碼更容易修改和使用，這種缺點通常是一個很小的代價。但是這種成本/效益分析在測試程式碼的背景下有一點不同。好的測試被設計成穩定的，事實上，當被測試的系統發生變化時，你通常希望它們會被破壞。因此，當涉及到測試程式碼時，DRY並沒有那麼多的好處。同時，對於測試來說，複雜性的成本更高：產品程式碼具有測試套件的優勢，可以確保它在變得複雜時繼續工作，而測試必須獨立進行，如果它們不明顯正確，則可能出現錯誤。如前所述，如果測試變得足夠複雜，以至於感覺需要自己的測試來確保它們正常工作，那麼就會出現問題。

Instead of being completely DRY, test code should often strive to be DAMP—that is, to promote “Descriptive And Meaningful Phrases.” A little bit of duplication is OK in tests so long as that duplication makes the test simpler and clearer. To illustrate, Example 12-19 presents some tests that are far too DRY.

與其說是完全的DRY，不如說測試程式碼應該經常努力做到DAMP——也就是提倡 "描述性和有意義的短語"。在測試中，一點點的重複是可以的，只要這種重複能使測試更簡單、更清晰。為了說明這一點，例12-19介紹了一些過於DRY的測試。

*Example 12-19. A test that is too DRY*   *例12-19. 一個過於DRY的測試*

```java
@Test
public void shouldAllowMultipleUsers() {
    List < User > users = createUsers(false, false);
    Forum forum = createForumAndRegisterUsers(users);
    validateForumAndUsers(forum, users);
}

@Test
public void shouldNotAllowBannedUsers() {
        List < User > users = createUsers(true);
        Forum forum = createForumAndRegisterUsers(users);
        validateForumAndUsers(forum, users);
}

// Lots more tests...
private static List < User > createUsers(boolean...banned) {
    List < User > users = new ArrayList < > ();
    for(boolean isBanned: banned) {
        users.add(newUser().setState(isBanned ? State.BANNED : State.NORMAL).build());
    }
    return users;
}

private static Forum createForumAndRegisterUsers(List < User > users) {
    Forum forum = new Forum();
    for(User user: users) {
        try {
            forum.register(user);
        } catch(BannedUserException ignored) {}
    }
    return forum;
}

private static void validateForumAndUsers(Forum forum, List < User > users) {
    assertThat(forum.isReachable()).isTrue();
    for(User user: users) {
        assertThat(forum.hasRegisteredUser(user)).isEqualTo(user.getState() == State.BANNED);
    }
}
```

The problems in this code should be apparent based on the previous discussion of clarity. For one, although the test bodies are very concise, they are not complete: important details are hidden away in helper methods that the reader can’t see without having to scroll to a completely different part of the file. Those helpers are also full of logic that makes them more difficult to verify at a glance (did you spot the bug?). The test becomes much clearer when it’s rewritten to use DAMP, as shown in Example 12-20.

基於前面對清晰度的討論，這段程式碼中的問題應該是顯而易見的。首先，儘管測試主體非常簡潔，但它們並不完整：重要的細節被隱藏在輔助方法中，讀者如果不滾動到檔案的完全不同部分就看不到這些方法。那些輔助方法也充滿了邏輯，使它們更難以一目瞭然地驗證（你發現了這個錯誤嗎？） 當它被改寫成使用DAMP時，測試就變得清晰多了，如例12-20所示。

*Example 12-20. Tests should be DAMP*   *例12-20. 測試應該是DAMP*

```java  
@Test
public void shouldAllowMultipleUsers() {
    User user1 = newUser().setState(State.NORMAL).build();
    User user2 = newUser().setState(State.NORMAL).build();

    Forum forum = new Forum();
    forum.register(user1);
    forum.register(user2);

    assertThat(forum.hasRegisteredUser(user1)).isTrue();
    assertThat(forum.hasRegisteredUser(user2)).isTrue();
}

@Test
public void shouldNotRegisterBannedUsers() {
    User user = newUser().setState(State.BANNED).build();

    Forum forum = new Forum();
    try {
        forum.register(user);
    } catch(BannedUserException ignored) {}

    assertThat(forum.hasRegisteredUser(user)).isFalse();
}
```

These tests have more duplication, and the test bodies are a bit longer, but the extra verbosity is worth it. Each individual test is far more meaningful and can be understood entirely without leaving the test body. A reader of these tests can feel confident that the tests do what they claim to do and aren’t hiding any bugs.

這些測試有更多的重複，測試體也有點長，但額外的言辭是值得的。每個單獨的測試都更有意義，不離開測試主體就可以完全理解。這些測試的讀者可以確信，這些測試做了他們聲稱要做的事情，並且沒有隱藏任何bug。

DAMP is not a replacement for DRY; it is complementary to it. Helper methods and test infrastructure can still help make tests clearer by making them more concise, factoring out repetitive steps whose details aren’t relevant to the particular behavior being tested. The important point is that such refactoring should be done with an eye toward making tests more descriptive and meaningful, and not solely in the name of reducing repetition. The rest of this section will explore common patterns for sharing code across tests.

DAMP不是DRY的替代品；它是對DRY的補充。輔助方法和測試基礎設施仍然可以幫助使測試更清晰，使其更簡潔，剔除重複的步驟，其細節與被測試的特定行為不相關。重要的一點是，這樣的重構應該著眼於使測試更有描述性和意義，而不是僅僅以減少重複的名義進行。本節的其餘部分將探討跨測試共享程式碼的常見模式。

### Shared  Values  共享值

Many tests are structured by defining a set of shared values to be used by tests and then by defining the tests that cover various cases for how these values interact. Example 12-21 illustrates what such tests look like.

許多測試的結構是透過定義一組測試使用的共享值，然後透過定義測試來涵蓋這些值如何互動的各種情況。例12-21說明了此類別測試的模樣。

*Example 12-21. Shared values with ambiguous names*  *例12-21. 名稱不明確的共享值*

```java
private static final Account ACCOUNT_1 = Account.newBuilder()
    .setState(AccountState.OPEN).setBalance(50).build();

private static final Account ACCOUNT_2 = Account.newBuilder()
    .setState(AccountState.CLOSED).setBalance(0).build();

private static final Item ITEM = Item.newBuilder()
    .setName("Cheeseburger").setPrice(100).build();

// Hundreds of lines of other tests...

@Test
public void canBuyItem_returnsFalseForClosedAccounts() {
    assertThat(store.canBuyItem(ITEM, ACCOUNT_1)).isFalse();
}

@Test
public void canBuyItem_returnsFalseWhenBalanceInsufficient() {
    assertThat(store.canBuyItem(ITEM, ACCOUNT_2)).isFalse();
}
```

This strategy can make tests very concise, but it causes problems as the test suite grows. For one, it can be difficult to understand why a particular value was chosen for a test. In Example 12-21, the test names fortunately clarify which scenarios are being tested, but you still need to scroll up to the definitions to confirm that ACCOUNT_1 and ACCOUNT_2 are appropriate for those scenarios. More descriptive constant names (e.g.,CLOSED_ACCOUNT and ACCOUNT_WITH_LOW_BALANCE) help a bit, but they still make it more difficult to see the exact details of the value being tested, and the ease of reusing these values can encourage engineers to do so even when the name doesn’t exactly describe what the test needs.

此策略可以使測試非常簡潔，但隨著測試套件的增長，它會導致問題。首先，很難理解為什麼選擇某個特定值進行測試。在示例12-21中，幸運的是，測試名稱澄清了正在測試的場景，但你仍然需要向上滾動到定義，以確認ACCOUNT_1和ACCOUNT_2適用於這些場景。更具描述性的常量名稱（例如，CLOSED_ACCOUNT 和 ACCOUNT_WITH_LOW_BALANCE）有一些幫助，但它們仍然使檢視被測試值的確切細節變得更加困難，並且重用這些值的方便性可以鼓勵工程師這樣做，即使名稱不能準確描述測試需要什麼。

Engineers are usually drawn to using shared constants because constructing individual values in each test can be verbose. A better way to accomplish this goal is to construct data using helper methods (see Example 12-22) that require the test author to specify only values they care about, and setting reasonable defaults7 for all other values. This construction is trivial to do in languages that support named parameters, but languages without named parameters can use constructs such as the Builder pattern to emulate them (often with the assistance of tools such as AutoValue):

工程師通常傾向於使用共享常量，因為在每個測試中構造單獨的值可能會很冗長。實現此目標的更好方法是使用輔助方法（參見示例12-22）構造資料，該方法要求測試作者僅指定他們關心的值，並為所有其他值設定合理的預設值。在支援命名引數的語言中，這種構造非常簡單，但是沒有命名引數的語言可以使用建構器模式等構造來模擬它們（通常需要AutoValue等工具的幫助）：

*Example 12-22. Shared values using helper methods*   *例12-22. 使用輔助方法的共享值*

```java
# A helper method wraps a constructor by defining arbitrary defaults for 
# each of its parameters.
def newContact(
        firstName = "Grace", lastName = "Hopper", phoneNumber = "555-123-4567"):
    return Contact(firstName, lastName, phoneNumber)

# Tests call the helper, specifying values for only the parameters that they
# care about.
def test_fullNameShouldCombineFirstAndLastNames(self):
    def contact = newContact(firstName = "Ada", lastName = "Lovelace") self.assertEqual(contact.fullName(), "Ada Lovelace")

// Languages like Java that don’t support named parameters can emulate them
// by returning a mutable "builder" object that represents the value under
// construction.
private static Contact.Builder newContact() {
    return Contact.newBuilder()
        .setFirstName("Grace")
        .setLastName("Hopper")
        .setPhoneNumber("555-123-4567");
}

// Tests then call methods on the builder to overwrite only the parameters
// that they care about, then call build() to get a real value out of the
// builder. @Test
public void fullNameShouldCombineFirstAndLastNames() {
    Contact contact = newContact()
        .setFirstName("Ada").setLastName("Lovelace")
        .build();
    assertThat(contact.getFullName()).isEqualTo("Ada Lovelace");
}
```

Using helper methods to construct these values allows each test to create the exact values it needs without having to worry about specifying irrelevant information or conflicting with other tests.

使用輔助方法來建構這些值，允許每個測試建立它所需要的精確值，而不必擔心指定不相關的資訊或與其他測試衝突。

> [^7]:	In many cases, it can even be useful to slightly randomize the default values returned for fields that aren’t explicitly set. This helps to ensure that two different instances won’t accidentally compare as equal, and makes it more difficult for engineers to hardcode dependencies on the defaults./
> 7 在許多情況下，甚至可以對未顯式設定的欄位返回的預設值進行輕微的隨機化。這有助於確保兩個不同的實例不會意外地比較為相等，並使工程師更難硬編碼對預設值的依賴關係。

### Shared Setup  共享設定

A related way that tests shared code is via setup/initialization logic. Many test frameworks allow engineers to define methods to execute before each test in a suite is run. Used appropriately, these methods can make tests clearer and more concise by obviating the repetition of tedious and irrelevant initialization logic. Used inappropriately, these methods can harm a test’s completeness by hiding important details in a separate initialization method.

測試共享程式碼的相關方法是透過設定/初始化邏輯。許多測試框架允許工程師在執行套件中的每個測試之前定義要執行的方法。如果使用得當，這些方法可以避免重複繁瑣和不相關的初始化邏輯，從而使測試更清晰、更簡潔。如果使用不當，這些方法會在單獨的初始化方法中隱藏重要細節，從而損害測試的完整性。

The best use case for setup methods is to construct the object under tests and its collaborators. This is useful when the majority of tests don’t care about the specific arguments used to construct those objects and can let them stay in their default states. The same idea also applies to stubbing return values for test doubles, which is a concept that we explore in more detail in Chapter 13.

設定方法的最佳用例是構造被測試物件及其合作者們。當大多數測試不關心用於構造這些物件的特定引數，並且可以讓它們保持預設狀態時，這非常有用。同樣的想法也適用於測試替換的打樁返回值，這是一個我們在第13章中詳細探討的概念。

One risk in using setup methods is that they can lead to unclear tests if those tests begin to depend on the particular values used in setup. For example, the test in Example 12-23 seems incomplete because a reader of the test needs to go hunting to discover where the string “Donald Knuth” came from.

使用設定方法的一個風險是，如果這些測試開始依賴於設定中使用的特定值，它們可能導致測試不明確。例如，例12-23中的測試似乎不完整，因為測試的讀者需要去尋找字串“Donald Knuth”的來源。

*Example 12-23. Dependencies on values in setup methods*   *例12-23. 設定方法中對數值的依賴性*

```java
private NameService nameService;
private UserStore userStore;

@Before
public void setUp() {
    nameService = new NameService();
    nameService.set("user1", "Donald Knuth");
    userStore = new UserStore(nameService);
}

// [... hundreds of lines of tests ...]

@Test
public void shouldReturnNameFromService() {
    UserDetails user = userStore.get("user1");
    assertThat(user.getName()).isEqualTo("Donald Knuth");
}
```

Tests like these that explicitly care about particular values should state those values directly, overriding the default defined in the setup method if need be. The resulting test contains slightly more repetition, as shown in Example 12-24, but the result is far more descriptive and meaningful.

像這樣明確關心特定值的測試應該直接說明這些值，如果需要的話，可以覆蓋setup方法中定義的預設值。如例12-24所示，所產生的測試包含了稍多的重複，但其結果是更有描述性和意義的。

*Example 12-24. Overriding values in setup Methods*   *例12-24. 重寫設定方法中的值*

```java
private NameService nameService;
private UserStore userStore;

@Before
public void setUp() {
    nameService = new NameService();
    nameService.set("user1", "Donald Knuth");
    userStore = new UserStore(nameService);
}

@Test
public void shouldReturnNameFromService() {
    nameService.set("user1", "Margaret Hamilton");
    UserDetails user = userStore.get("user1");
    assertThat(user.getName()).isEqualTo("Margaret Hamilton");
}
```

### Shared  Helpers  and  Validation  共享輔助和驗證

The last common way that code is shared across tests is via “helper methods” called from the body of the test methods. We already discussed how helper methods can be a useful way for concisely constructing test values—this usage is warranted, but other types of helper methods can be dangerous.

最後一種在測試中共享程式碼的常見方式是透過從測試方法主體中呼叫 "輔助方法"。我們已經討論了輔助方法如何成為簡明地建構測試值的有用方法——這種用法是有必要的，但其他型別的輔助方法可能是危險的。

One common type of helper is a method that performs a common set of assertions against a system under test. The extreme example is a validate method called at the end of every test method, which performs a set of fixed checks against the system under test. Such a validation strategy can be a bad habit to get into because tests using this approach are less behavior driven. With such tests, it is much more difficult to determine the intent of any particular test and to infer what exact case the author had in mind when writing it. When bugs are introduced, this strategy can also make them more difficult to localize because they will frequently cause a large number of tests to start failing.

一種常見的輔助工具是對被測系統執行一套共同的斷言的方法。極端的例子是在每個測試方法的末尾呼叫一個驗證方法，它對被測系統執行一組固定的檢查。這樣的驗證策略可能是一個不好的習慣，因為使用這種方法的測試是較少的行為驅動。有了這樣的測試，就更難確定任何特定測試的意圖，也更難推斷出作者在編寫測試時到底想到了什麼情況。當bug被引入時，這種策略也會使它們更難被定位，因為它們會經常導致大量的測試開始失敗。

More focused validation methods can still be useful, however. The best validation helper methods assert a single conceptual fact about their inputs, in contrast to general-purpose validation methods that cover a range of conditions. Such methods can be particularly helpful when the condition that they are validating is conceptually simple but requires looping or conditional logic to implement that would reduce clarity were it included in the body of a test method. For example, the helper method in Example 12-25 might be useful in a test covering several different cases around account access.

然而，更有針對性的驗證方法仍然是有用的。最好的驗證輔助方法只斷言其輸入的一個概念性事實，與涵蓋一系列條件的通用驗證方法相反。當他們驗證的條件在概念上很簡單，但需要迴圈或條件邏輯來實現，如果將其包含在測試方法的主體中，就會降低清晰度，這樣的方法特別有用。例如，例12-25中的輔助方法在測試中可能很有用，它涵蓋了圍繞賬戶訪問的幾種不同情況。

*Example 12-25. A conceptually simple test*   *例12-25. 概念上簡單的測試*

```java
private void assertUserHasAccessToAccount(User user, Account account) {
    for(long userId: account.getUsersWithAccess()) {
        if(user.getId() == userId) {
            return;
        }
    }
    fail(user.getName() + " cannot access " + account.getName());
}
```

### Defining Test Infrastructure  界定測試基礎框架
>>>>>>> e62a431152c49eed54adedf4695677544653c19f
The techniques we’ve discussed so far cover sharing code across methods in a single test class or suite. Sometimes, it can also be valuable to share code across multiple test suites. We refer to this sort of code as test infrastructure. Though it is usually more valuable in integration or end-to-end tests, carefully designed test infrastructure can make unit tests much easier to write in some circumstances.

到目前為止，我們討論的技術包括在單個測試類別或測試套件中跨方法共享程式碼。有時，跨多個測試套件共享程式碼也很有價值。我們將這種程式碼稱為測試基礎框架。儘管它通常在整合或端到端測試中更有價值，但精心設計的測試基礎框架可以使單元測試在某些情況下更易於編寫。

Custom test infrastructure must be approached more carefully than the code sharing that happens within a single test suite. In many ways, test infrastructure code is more similar to production code than it is to other test code given that it can have many callers that depend on it and can be difficult to change without introducing breakages. Most engineers aren’t expected to make changes to the common test infrastructure while testing their own features. Test infrastructure needs to be treated as its own separate product, and accordingly, test infrastructure must always have its own tests.

自訂測試基礎框架必須比在單個測試套件中發生的程式碼共享更謹慎地對待。在許多方面，測試基礎框架的程式碼比其他測試程式碼更類似於產品程式碼，因為它可能有許多依賴它的呼叫者，並且在不引入破壞的情況下很難改變。大多數工程師不應該在測試他們自己的功能時對通用測試基礎框架進行修改。測試基礎框架需要被當作自己獨立的產品，相應地，測試基礎框架必須始終有自己的測試。

Of course, most of the test infrastructure that most engineers use comes in the form of well-known third-party libraries like JUnit. A huge number of such libraries are available, and standardizing on them within an organization should happen as early and universally as possible. For example, Google many years ago mandated Mockito as the only mocking framework that should be used in new Java tests and banned new tests from using other mocking frameworks. This edict produced some grumbling at the time from people comfortable with other frameworks, but today, it’s universally seen as a good move that made our tests easier to understand and work with.

當然，大多數工程師使用的測試基礎框架都是以知名的第三方函式庫的形式出現的，如JUnit。有大量這樣的函式庫可以使用，在一個組織內對它們進行標準化應該儘可能早地和普遍地發生。例如，Google多年前規定Mockito是新的Java測試中唯一應該使用的模擬框架，並禁止新的測試使用其他模擬框架。這一規定在當時引起了一些對其他框架感到滿意的人的不滿，但今天，人們普遍認為這是一個好的舉措，使我們的測試更容易理解和使用。

## Conclusion

Unit tests are one of the most powerful tools that we as software engineers have to make sure that our systems keep working over time in the face of unanticipated changes. But with great power comes great responsibility, and careless use of unit testing can result in a system that requires much more effort to maintain and takes much more effort to change without actually improving our confidence in said system.

單元測試是我們作為軟體工程師所擁有的最強大的工具之一，它可以確保我們的系統在面對意料之外的變化時仍能正常工作。但是，強大的力量伴隨著巨大的責任，不小心使用單元測試會導致系統需要更多的努力來維護，需要更多的努力來更改，不然不會真正提高我們對所述系統的信心。

Unit tests at Google are far from perfect, but we’ve found tests that follow the practices outlined in this chapter to be orders of magnitude more valuable than those that don’t. We hope they’ll help you to improve the quality of your own tests!

谷歌的單元測試遠非完美，但我們發現遵循本章所述做法的測試比那些不遵循的測試要有價值得多。我們希望它們能幫助你提高你自己的測試的品質。

## TL;DRs  內容提要

- Strive for unchanging tests.

- Test via public APIs.

- Test state, not interactions.

- Make your tests complete and concise.

- Test behaviors, not methods.

- Structure tests to emphasize behaviors.

- Name tests after the behavior being tested.

- Don’t put logic in tests.

- Write clear failure messages.

- Follow DAMP over DRY when sharing code for tests.

- 努力實現穩定的測試。

- 透過公共API進行測試。

- 測試狀態，而不是互動。

- 使你的測試完整和簡明。

- 測試行為，而不是方法。

- 強調行為的結構測試。

- 使用被測試的行為來命名測試。

- 不要把邏輯放在測試中。

- 編寫清晰的失敗資訊。

- 在共享測試的程式碼時，遵循DAMP而不是DRY。

