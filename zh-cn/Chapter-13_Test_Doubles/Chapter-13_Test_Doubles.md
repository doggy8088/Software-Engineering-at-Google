**CHAPTER 13**

# Test Doubles

# 第十三章 測試替代

**Written by Andrew Trenk and Dillon Bly**

**Edited by Tom Manshreck**

Unit tests are a critical tool for keeping developers productive and reducing defects in code. Although they can be easy to write for simple code, writing them becomes difficult as code becomes more complex.

單元測試是保持開發人員生產力和減少程式碼缺陷的重要工具。儘管對於簡單的程式碼來說，單元測試很容易編寫，但當代碼變得更加複雜時，編寫單元測試就變得困難了。

For example, imagine trying to write a test for a function that sends a request to an external server and then stores the response in a database. Writing a handful of tests might be doable with some effort. But if you need to write hundreds or thousands of tests like this, your test suite will likely take hours to run, and could become flaky due to issues like random network failures or tests overwriting one another’s data.

例如，想象一下，嘗試為一個函式編寫測試，該函式向外部伺服器傳送請求，然後將響應儲存在資料庫中。只需多付出一些努力，編寫少量的測試可能是可以做到的。但如果你需要寫成百上千個這樣的測試，你的測試套件很可能需要幾個小時才能執行，並且可能由於隨機網路故障或測試相互覆蓋資料等問題讓測試變得不穩定。

Test doubles come in handy in such cases. A test double is an object or function that can stand in for a real implementation in a test, similar to how a stunt double can stand in for an actor in a movie. The use of test doubles is often referred to as mocking, but we avoid that term in this chapter because, as we’ll see, that term is also used to refer to more specific aspects of test doubles.

在這種情況下，測試替代就會派上用場。測試替代可以是一個物件或函式，它可以在測試中代替真正的實現，類似於特技替身可以代替電影中的演員那樣。測試替代的使用通常被稱為模擬，但我們在本章中避免使用這個術語，因為正如我們將看到的，這個術語也被用來指代測試替代的更具體方面。

Perhaps the most obvious type of test double is a simpler implementation of an object that behaves similarly to the real implementation, such as an in-memory database. Other types of test doubles can make it possible to validate specific details of your system, such as by making it easy to trigger a rare error condition, or ensuring a heavyweight function is called without actually executing the function’s implementation.

也許最明顯的測試替代型別是一個行為類似於真實實現的物件的更簡單的實現，比如一個記憶體資料庫。其他型別的測試替代可以驗證系統的特定細節，例如透過使觸發罕見錯誤條件變得容易，或者確保在不實際執行函式實現的情況下呼叫重量級函式。

The previous two chapters introduced the concept of small tests and discussed why they should comprise the majority of tests in a test suite. However, production code often doesn’t fit within the constraints of small tests due to communication across multiple processes or machines. Test doubles can be much more lightweight than real implementations, allowing you to write many small tests that execute quickly and are not flaky.

前兩章介紹了小型測試的概念，並討論了為什麼它們應該包括測試套件中的大多數測試。然而，由於跨多個程序或機器的通訊，生產程式碼往往不適合小型測試的約束。測試替代可以比真正的實現更輕量級，允許你寫許多小測試，快速執行，並且不易出錯。

## The Impact of Test Doubles on Software Development  測試替代對軟體開發的影響
The use of test doubles introduces a few complications to software development that require some trade-offs to be made. The concepts introduced here are discussed in more depth throughout this chapter:

*Testability*  
	To use test doubles, a codebase needs to be designed to be testable—it should be possible for tests to swap out real implementations with test doubles. For example, code that calls a database needs to be flexible enough to be able to use a test double in place of a real database. If the codebase isn’t designed with testing in mind and you later decide that tests are needed, it can require a major commitment to refactor the code to support the use of test doubles.

*Applicability*  
	Although proper application of test doubles can provide a powerful boost to engineering velocity, their improper use can lead to tests that are brittle, complex, and less effective. These downsides are magnified when test doubles are used improperly across a large codebase, potentially resulting in major losses in productivity for engineers. In many cases, test doubles are not suitable and engineers should prefer to use real implementations instead.

*Fidelity*  
	Fidelity refers to how closely the behavior of a test double resembles the behavior of the real implementation that it’s replacing. If the behavior of a test double significantly differs from the real implementation, tests that use the test double likely wouldn’t provide much value—for example, imagine trying to write a test with a test double for a database that ignores any data added to the database and always returns empty results. But perfect fidelity might not be feasible; test doubles often need to be vastly simpler than the real implementation in order to be suitable for use in tests. In many situations, it is appropriate to use a test double even without perfect fidelity. Unit tests that use test doubles often need to be supplemented by larger-scope tests that exercise the real implementation.

測試替代的使用給軟體開發帶來了一些複雜的問題，需要做出一些權衡。本章將更深入地討論此處介紹的概念：

*可測試性*  
	為了使用測試替代，需要將程式碼函式庫設計成可測試的--測試應該可以用測試替代替換實際實現。例如，呼叫資料庫的程式碼需要足夠靈活，以便能夠使用測試替代來代替真正的資料庫。如果程式碼函式庫在設計時沒有考慮到測試，而你後來決定需要測試，那麼可能需要進行大量的提交來重構程式碼，以支援使用測試替代。

*適用性*  
	儘管適當地應用測試替代可以極大地提高工程速度，但其使用不當會導致測試變得脆弱、複雜且低效。當測試替代在大型程式碼函式庫中使用不當時，這些缺點就會被放大，這可能會導致工程師在生產效率方面的重大損失。在許多情況下，測試替代是不合適的，工程師應該傾向於使用真實的實現。

*模擬度*  
	模擬度是指測試替代的行為與它所替代的真實實現的行為有多大的相似性。如果測試替代的行為與真正的實現有很大的不同，那麼使用測試替代的測試可能不會提供太多的價值——例如，想象一下，嘗試用測試替代為一個數據函式庫寫一個測試，這個資料庫忽略了新增到資料庫的任何資料，總是返回空結果。這樣做是完美的模擬度不能接受的；測試替代通常需要比實際的實現簡單得多，以便適合在測試中使用。在許多情況下，即使沒有完美的模擬度，使用測試替代也是合適的。使用測試替代的單元測試通常需要由執行實際實現的更大範圍的測試來支援。

## Test Doubles at Google 谷歌的測試替代

At Google, we’ve seen countless examples of the benefits to productivity and software quality that test doubles can bring to a codebase, as well as the negative impact they can cause when used improperly. The practices we follow at Google have evolved over time based on these experiences. Historically, we had few guidelines on how to effectively use test doubles, but best practices evolved as we saw common patterns and antipatterns arise in many teams’ codebases.

在谷歌，我們已經看到了無數的例子，證明測試替代可以為程式碼函式庫提升生產力和軟體品質方面的好處，以及在使用不當時可能造成的負面影響。我們在谷歌遵循的做法是基於這些經驗隨著時間的推移而演變的。從歷史上看，我們很少有關於如何有效地使用測試替代，但最佳實踐隨著我們看到許多團隊的程式碼函式庫中出現了常見模式和反模式而不斷髮展。

One lesson we learned the hard way is the danger of overusing mocking frameworks, which allow you to easily create test doubles (we will discuss mocking frameworks in more detail later in this chapter). When mocking frameworks first came into use at Google, they seemed like a hammer fit for every nail—they made it very easy to write highly focused tests against isolated pieces of code without having to worry about how to construct the dependencies of that code. It wasn’t until several years and countless tests later that we began to realize the cost of such tests: though these tests were easy to write, we suffered greatly given that they required constant effort to maintain while rarely finding bugs. The pendulum at Google has now begun swinging in the other direction, with many engineers avoiding mocking frameworks in favor of writing more realistic tests.

我們經過艱苦的歷程學到的一個教訓是過度使用模擬框架的危險，它允許你輕鬆建立測試替代（我們將在本章後面更詳細地討論模擬框架）。當mocking框架首次在Google使用時，它們就像一把錘子，適合每一根釘子。它們使得針對獨立的程式碼段編寫高度集中的測試變得非常容易，而不必擔心如何建構程式碼的依賴關係。直到經過幾年和無數次測試之後，我們才開始意識到這些測試的成本：儘管這些測試很容易編寫，但由於它們需要不斷的努力來維護，而很少發現bug，我們遭受了巨大的損失。谷歌的天平現在開始向另一個方向擺動，許多工程師避免mocking框架，轉而編寫更真實的測試。

Even though the practices discussed in this chapter are generally agreed upon at Google, the actual application of them varies widely from team to team. This variance stems from engineers having inconsistent knowledge of these practices, inertia in an existing codebase that doesn’t conform to these practices, or teams doing what is easiest for the short term without thinking about the long-term implications.

儘管本章中討論的實踐在谷歌公司得到普遍認可，但實際應用情況因團隊而異。這種差異源於工程師對這些實踐的認識差異，現有程式碼函式庫中的習慣不符合這些實踐，或者團隊只做短期內最容易的事情而不考慮長期影響。

## Basic Concepts 基本概念

Before we dive into how to effectively use test doubles, let’s cover some of the basic concepts related to them. These build the foundation for best practices that we will discuss later in this chapter.

在我們深入研究如何有效地使用測試替代之前，讓我們先介紹一些與之相關的基本概念。這些為我們在本章後面討論的最佳實踐奠定了基礎。

### An Example Test Double 測試替代的示例

Imagine an ecommerce site that needs to process credit card payments. At its core, it might have something like the code shown in [Example 13-1](#_bookmark1068).

想象一個需要處理信用卡支付的電子商務網站。在其核心部分，它可能有類似於例13-1中所示的程式碼。

*Example* *13-1.* *A* *credit* *card* *service*

```java
class PaymentProcessor {
  private CreditCardService creditCardService;
  ...
  boolean makePayment(CreditCard creditCard, Money amount) {
    if (creditCard.isExpired()) { return false; }
    boolean success = creditCardService.chargeCreditCard(creditCard,  amount);
    return success;
  }
}

```

It would be infeasible to use a real credit card service in a test (imagine all the transaction fees from running the test!), but a test double could be used in its place to *simulate* the behavior of the real system. The code in [Example 13-2](#_bookmark1069) shows an extremely simple test double.

在測試中使用真正的信用卡服務是不可行的（想象一下執行測試所產生的所有交易費用！），但是可以用一個測試用的替代來*模擬*真實系統的行為。例13-2中的程式碼展示了一個非常簡單的測試替代。

*Example 13-2. A trivial test double*

```java
class TestDoubleCreditCardService implements CreditCardService {
  @Override
  public boolean chargeCreditCard(CreditCard creditCard, Money amount) {
    return true;
  }
}
```

Although this test double doesn’t look very useful, using it in a test still allows us to test some of the logic in the makePayment() method. For example, in [Example 13-3](#_bookmark1070), we can validate that the method behaves properly when the credit card is expired because the code path that the test exercises doesn’t rely on the behavior of the credit card service.

雖然這個測試替代看起來不是很有用，但在測試中使用它仍然可以讓我們測試makePayment()方法中的一些邏輯。例如，在例13-3中，我們可以驗證該方法在信用卡過期時的行為是否正確，因為測試執行的程式碼路徑不依賴於信用卡服務的行為。

*Example 13-3. Using the test double*

```java
@Test public void cardIsExpired_returnFalse() {
	boolean success = paymentProcessor.makePayment(EXPIRED_CARD, AMOUNT);
    assertThat(success).isFalse();
}
```

The following sections in this chapter will discuss how to make use of test doubles in more complex situations than this one.

本章下面的章節將討論如何在比這更復雜的情況下使用測試替代。

### Seams

```
Seams是可以更改程式中的行為而無需在指定位置進行編輯的地方。
```

Code is said to be [*testable* ](https://oreil.ly/yssV2)if it is written in a way that makes it possible to write unit tests for the code. A [*seam* ](https://oreil.ly/pFSFf)is a way to make code testable by allowing for the use of test doubles—it makes it possible to use different dependencies for the system under test rather than the dependencies used in a production environment.

如果程式碼的編寫方式能夠使程式碼的單元測試成為可能，那麼程式碼就被稱為[*可測試程式碼*](https://oreil.ly/yssV2)。[*seam*](https://oreil.ly/pFSFf)是一種透過允許使用測試替代使程式碼可測試的方法——它使被測系統可以使用不同的依賴項，而不是生產環境中使用的依賴項。

[*Dependency* *injection* ](https://oreil.ly/og9p9)is a common technique for introducing seams. In short, when a class utilizes dependency injection, any classes it needs to use (i.e., the class’s *dependencies*) are passed to it rather than instantiated directly, making it possible for these dependencies to be substituted in tests.

[*依賴注入*](https://oreil.ly/og9p9)是一種引入seams的常見技術。簡而言之，當一個類別利用依賴注入時，它需要使用的任何類別（即該類別的*依賴*）被傳遞給它，而不是直接實例化，從而使這些依賴項可以在測試中被替換。

[Example 13-4 ](#_bookmark1074)shows an example of dependency injection. Rather than the constructor creating an instance of CreditCardService, it accepts an instance as a parameter.

示例13-4顯示了依賴項注入的示例。它接受實例作為引數，而不是建立CreditCardService實例的建構函式。

*Example* *13-4.* *Dependency* *injection*

```java
class PaymentProcessor {
  private CreditCardService creditCardService;

  PaymentProcessor(CreditCardService creditCardService) {
    this.creditCardService = creditCardService;
  }
  ...
}

```

The code that calls this constructor is responsible for creating an appropriate Credit CardService instance. Whereas the production code can pass in an implementation of CreditCardService that communicates with an external server, the test can pass in a test double, as demonstrated in [Example 13-5](#_bookmark1075).

呼叫這個建構函式的程式碼負責建立一個合適的CreditCardService實例。生產程式碼可以傳入一個與外部伺服器通訊的CreditCardService的實現，而測試可以傳入一個測試用的替代，如例13-5所示。

*Example 13-5. Passing in a test double*

```java
class PaymentProcessor {
  private CreditCardService creditCardService;

  PaymentProcessor(CreditCardService creditCardService) {
    this.creditCardService = creditCardService;
  }
  ...
}
```

To reduce boilerplate associated with manually specifying constructors, automated dependency injection frameworks can be used for constructing object graphs automatically. At Google, [Guice ](https://github.com/google/guice)and [Dagger ](https://google.github.io/dagger)are automated dependency injection frameworks that are commonly used for Java code.

為了減少與手動指定建構函式有關的範本，可以使用自動依賴注入框架來自動建構物件。在谷歌，[Guice](https://github.com/google/guice)和[Dagger](https://google.github.io/dagger)是自動依賴注入框架，通常用於Java程式碼。

With dynamically typed languages such as Python or JavaScript, it is possible to dynamically replace individual functions or object methods. Dependency injection is less important in these languages because this capability makes it possible to use real implementations of dependencies in tests while only overriding functions or methods of the dependency that are unsuitable for tests.

對於動態型別的語言，如Python或JavaScript，有可能動態地替換單個函式或物件方法。依賴注入在這些語言中不太重要，因為這種功能使得在測試中使用依賴項的真實實現成為可能，同時只覆蓋不適合測試的依賴項的函式或方法。

Writing testable code requires an upfront investment. It is especially critical early in the lifetime of a codebase because the later testability is taken into account, the more difficult it is to apply to a codebase. Code written without testing in mind typically needs to be refactored or rewritten before you can add appropriate tests.

編寫可測試程式碼需要前期投入。在程式碼函式庫生命週期的早期，這一點尤其重要，因為越晚考慮可測試性，就越難應用到程式碼函式庫中。在沒有考慮到測試的情況下編寫的程式碼通常需要重構或重寫，然後才可以新增適當的測試。

### Mocking Frameworks 模擬框架

A *mocking framework* is a software library that makes it easier to create test doubles within tests; it allows you to replace an object with a *mock*, which is a test double whose behavior is specified inline in a test. The use of mocking frameworks reduces boilerplate because you don’t need to define a new class each time you need a test double.

一個*mocking框架*是一個軟體函式庫，它使得在測試中建立測試替代更加容易；它允許您將物件替換為模擬物件，模擬物件是在測試中內聯指定其行為的測試替代。模擬框架的使用減少了範本程式碼，因為你不需要在每次需要測試時定義一個新類別。

[Example 13-6](#_bookmark1081) demonstrates the use of [Mockito](https://site.mockito.org/), a mocking framework for Java. Mockito creates a test double for CreditCardService and instructs it to return a specific value.

 例13-6示範了[Mockito](https://site.mockito.org/)的使用，這是一個Java的模擬框架。Mockito為CreditCardService建立了一個測試替代，並指定它返回一個特定的值。

*Example 13-6. Mocking frameworks*

```java
class PaymentProcessorTest {
...
PaymentProcessor paymentProcessor;

// Create a test double of CreditCardService with just one line of code.
@Mock CreditCardService mockCreditCardService;

@Before public void setUp() {
    // Pass in the test double to the system under test.
    paymentProcessor = new PaymentProcessor(mockCreditCardService);
}

@Test public void chargeCreditCardFails_returnFalse() {
    // Give some behavior to the test double: it will return false
    // anytime the chargeCreditCard() method is called. The usage of
    // “any()” for the method’s arguments tells the test double to
    // return false regardless of which arguments are passed.
    when(mockCreditCardService.chargeCreditCard(any(), any())
    	.thenReturn(false);
    boolean success = paymentProcessor.makePayment(CREDIT_CARD, AMOUNT);
    assertThat(success).isFalse();
  }
}
```

Mocking frameworks exist for most major programming languages. At Google, we use Mockito for Java, [the googlemock component of Googletest ](https://github.com/google/googletest)for C++, and [uni‐](https://oreil.ly/clzvH) [ttest.mock ](https://oreil.ly/clzvH)for Python.

大多數主要的程式語言都有模擬框架。在Google，我們在Java中使用Mockito，在C++中使用[Googletest的googlemock元件](https://github.com/google/googletest)，在Python中使用[uni-ttest.mock](https://oreil.ly/clzvH) 。

Although mocking frameworks facilitate easier usage of test doubles, they come with some significant caveats given that their overuse will often make a codebase more difficult to maintain. We cover some of these problems later in this chapter.

儘管模擬框架有助於更容易地使用測試替代，但它們也有一些重要的注意事項，因為過度使用它們往往會使程式碼函式庫更難維護。我們將在本章的後面介紹其中的一些問題。

## Techniques for Using Test Doubles 測試替代的使用技巧

There are three primary techniques for using test doubles. This section presents a brief introduction to these techniques to give you a quick overview of what they are and how they differ. Later sections in this chapter go into more details on how to effectively apply them.

使用測試替代有三種主要技術。本節簡要介紹這些技術，讓您快速瞭解它們是什麼以及它們之間的區別。本章後面幾節將詳細介紹如何有效地應用它們。

An engineer who is aware of the distinctions between these techniques is more likely to know the appropriate technique to use when faced with the need to use a test double.

知道到這些技術之間區別的工程師更有可能在面臨需要使用測試替代時知道使用哪種適當的技術。

### Faking 偽造

A [*fake*](https://oreil.ly/rymnI) is a lightweight implementation of an API that behaves similar to the real implementation but isn’t suitable for production; for example, an in-memory database. [Example 13-7 ](#_bookmark1089)presents an example of faking.

 [*fake*](https://oreil.ly/rymnI)是一個API的輕量級實現，其行為類似於真實實現，但不適合生產；例如，一個記憶體資料庫。例13-7介紹了一個偽造的例子。

*Example 13-7. A simple* *fake*

```java
// Creating the fake is fast and easy.
AuthorizationService fakeAuthorizationService = new FakeAuthorizationService();
AccessManager accessManager = new AccessManager(fakeAuthorizationService):

// Unknown user IDs shouldn’t have access.
assertFalse(accessManager.userHasAccess(USER_ID));

// The user ID should have access after it is added to
// the authorization service. 
fakeAuthorizationService.addAuthorizedUser(new User(USER_ID));
assertThat(accessManager.userHasAccess(USER_ID)).isTrue();

```

Using a fake is often the ideal technique when you need to use a test double, but a fake might not exist for an object you need to use in a test, and writing one can be challenging because you need to ensure that it has similar behavior to the real implementation, now and in the future.

當你需要使用測試替代時，使用偽造通常是理想的技術，但是對於你需要在測試中使用的物件，偽造可能不存在，編寫偽造可能是一項挑戰，因為你需要確保它在現在和將來具有與真實實現類似的行為。

### Stubbing 打樁

[*Stubbing* ](https://oreil.ly/gmShS)is the process of giving behavior to a function that otherwise has no behavior on its own—you specify to the function exactly what values to return (that is, you *stub* the return values).

打樁是指將行為賦予一個函式的過程，如果該函式本身沒有行為，則你可以為該函式指定要返回的值（即打樁返回值）。

[Example 13-8](#_bookmark1093) illustrates stubbing. The when(...).thenReturn(...) method calls from the Mockito mocking framework specify the behavior of the lookupUser() method.

例13-8示範了打樁的使用。來自Mockito模擬框架的when(...).thenReturn(...)方法呼叫指定了lookupUser()方法的行為。

*Example* *13-8.* *Stubbing*

```java
// Pass in a test double that was created by a mocking framework.
AccessManager accessManager = new AccessManager(mockAuthorizationService):

// The user ID shouldn’t have access if null is returned. 
when(mockAuthorizationService.lookupUser(USER_ID)).thenReturn(null);
assertThat(accessManager.userHasAccess(USER_ID)).isFalse();

// The user ID should have access if a non-null value is returned.   
when(mockAuthorizationService.lookupUser(USER_ID)).thenReturn(USER);
assertThat(accessManager.userHasAccess(USER_ID)).isTrue();
```

Stubbing is typically done through mocking frameworks to reduce boilerplate that would otherwise be needed for manually creating new classes that hardcode return values.

打樁通常是透過模擬框架來完成的，以減少手動建立新的類別來硬編碼返回值所需的範本。

Although stubbing can be a quick and simple technique to apply, it has limitations, which we’ll discuss later in this chapter.

雖然打樁是一種快速而簡單的應用技術，但它也有侷限性，我們將在本章後面討論。

### Interaction Testing 互動測試

[*Interaction testing* ](https://oreil.ly/zGfFn)is a way to validate *how* a function is called without actually calling the implementation of the function. A test should fail if a function isn’t called the correct way—for example, if the function isn’t called at all, it’s called too many times, or it’s called with the wrong arguments.

互動測試是一種在不實際呼叫函式實現的情況下驗證函式呼叫方式的方法。如果函式沒有正確呼叫，測試應該失敗。例如，如果函式根本沒有被呼叫，呼叫次數太多，或者呼叫引數錯誤。

[Example 13-9 ](#_bookmark1097)presents an instance of interaction testing. The verify(...) method from the Mockito mocking framework is used to validate that lookupUser() is called as expected.

例13-9展示了一個互動測試的實例。來自Mockito 模擬框架的verify(...)方法被用來驗證lookupUser()是否按預期呼叫。

*Example* *13-9. Interaction testing*

```java
// Pass in a test double that was created by a mocking framework.
AccessManager accessManager = new AccessManager(mockAuthorizationService);
accessManager.userHasAccess(USER_ID);

// The test will fail if accessManager.userHasAccess(USER_ID) didn’t call
// mockAuthorizationService.lookupUser(USER_ID).
verify(mockAuthorizationService).lookupUser(USER_ID);

```

Similar to stubbing, interaction testing is typically done through mocking frameworks. This reduces boilerplate compared to manually creating new classes that contain code to keep track of how often a function is called and which arguments were passed in.

與打樁類似，互動測試通常是透過模擬框架完成的。與手動建立包含程式碼的新類別以追蹤函式呼叫頻率和傳入引數相比，這減少了範本檔案。

Interaction testing is sometimes called [*mocking*](https://oreil.ly/IfMoR). We avoid this terminology in this chapter because it can be confused with mocking frameworks, which can be used for stubbing as well as for interaction testing.

互動測試有時被稱為模擬。我們在本章中避免使用這個術語，因為它可能與模擬框架混淆，模擬框架既可用於打樁，也可用於互動測試。

As discussed later in this chapter, interaction testing is useful in certain situations but should be avoided when possible because overuse can easily result in brittle tests.

正如本章後面所討論的，互動測試在某些情況下很有用，但應儘可能避免，因為過度使用很容易導致脆性測試。

### Real Implementations 真實實現

Although test doubles can be invaluable testing tools, our first choice for tests is to use the real implementations of the system under test’s dependencies; that is, the same implementations that are used in production code. Tests have higher fidelity when they execute code as it will be executed in production, and using real implementations helps accomplish this.

儘管測試替代是非常有價值的測試工具，但我們對測試的第一選擇是使用被測系統依賴的真實實現；也就是說，與生產程式碼中使用的實現相同。當測試執行程式碼時，其模擬度更高，因為它將在生產中執行，使用真實實現有助於實現這一目標。

At Google, the preference for real implementations developed over time as we saw that overuse of mocking frameworks had a tendency to pollute tests with repetitive code that got out of sync with the real implementation and made refactoring difficult. We’ll look at this topic in more detail later in this chapter.

在谷歌，對真實實現的偏好隨著時間的推移而發展，因為我們看到過度使用模擬框架有一種傾向，即使用與真實實現不同步的重複程式碼汙染測試，從而使重構變得困難。我們將在本章後面更詳細地討論這個主題。

Preferring real implementations in tests is known as [*classical testing*](https://oreil.ly/OWw7h). There is also a style of testing known as *mockist testing*, in which the preference is to use mocking frameworks instead of real implementations. Even though some people in the software industry practice mockist testing (including the [creators of the first mocking](https://oreil.ly/_QWy7) [frameworks](https://oreil.ly/_QWy7)), at Google, we have found that this style of testing is difficult to scale. It requires engineers to follow [strict guidelines when designing the system under test](http://jmock.org/oopsla2004.pdf), and the default behavior of most engineers at Google has been to write code in a way that is more suitable for the classical testing style.

在測試中更傾向於使用真實實現被稱為[*經典測試*](https://oreil.ly/OWw7h)。還有一種測試風格被稱為*模擬測試*，其中傾向於使用模擬框架而不是真實實現。儘管軟體行業的一些人在進行模擬測試（包括[第一個模擬框架](https://oreil.ly/_QWy7)的創造者），但在谷歌，我們發現這種測試風格很難擴充套件。它要求工程師遵循[設計被測系統時的嚴格準則](http://jmock.org/oopsla2004.pdf)，而谷歌大多數工程師的預設行為是以一種更適合經典測試風格的方式來編寫程式碼。

### Prefer Realism Over Isolation 傾向於現實主義而不是孤立主義

Using real implementations for dependencies makes the system under test more realistic given that all code in these real implementations will be executed in the test. In contrast, a test that utilizes test doubles isolates the system under test from its dependencies so that the test does not execute code in the dependencies of the system under test.

考慮到這些真實實現中的所有程式碼都將在測試中執行，使用真實實現進行依賴測試會使被測系統更加真實。相比之下，使用測試替代的測試會將被測系統與其依賴隔離開來，這樣測試就不會在被測系統的依賴中執行程式碼。

We prefer realistic tests because they give more confidence that the system under test is working properly. If unit tests rely too much on test doubles, an engineer might need to run integration tests or manually verify that their feature is working as expected in order to gain this same level of confidence. Carrying out these extra tasks can slow down development and can even allow bugs to slip through if engineers skip these tasks entirely when they are too time consuming to carry out compared to running unit tests.

我們更喜歡真實測試，因為它們能讓人對被測系統的正常工作更有信心。如果單元測試過於依賴測試替代，工程師可能需要執行整合測試或手動驗證他們的功能是按預期工作的，以獲得同樣的信心水平。執行這些額外的任務會減慢開發速度，如果工程師完全跳過這些任務，那麼與執行單元測試相比，執行這些任務太耗時，甚至會讓bug溜走。

Replacing all dependencies of a class with test doubles arbitrarily isolates the system under test to the implementation that the author happens to put directly into the class and excludes implementation that happens to be in different classes. However, a good test should be independent of implementation—it should be written in terms of the API being tested rather than in terms of how the implementation is structured.

將一個類別的所有依賴項替換為測試替代項可以任意地將被測系統與作者直接放入類別中的實現隔離開來，並排除恰好位於不同類別中的實現。然而，一個好的測試應該獨立於實現，它應該根據API編寫正在進行測試，而不是根據實現的結構進行測試。

Using real implementations can cause your test to fail if there is a bug in the real implementation. This is good! You *want* your tests to fail in such cases because it indicates that your code won’t work properly in production. Sometimes, a bug in a real implementation can cause a cascade of test failures because other tests that use the real implementation might fail, too. But with good developer tools, such as a Continuous Integration (CI) system, it is usually easy to track down the change that caused the failure.

如果真實實現中存在錯誤，使用真實的實現會導致你的測試失敗。這是很好的。你希望你的測試在這種情況下失敗，因為它表明你的程式碼在生產中不能正常工作。有時，真實實現中的一個錯誤會導致一連串的測試失敗，因為其他使用真實實現的測試也可能失敗。但是有了好的開發者工具，如持續整合（CI）系統，通常很容易追蹤到導致失敗的變化。

-----

#### Case Study: @DoNotMock 案例研究：@DoNotMock

At Google, we’ve seen enough tests that over-rely on mocking frameworks to motivate the creation of the @DoNotMock annotation in Java, which is available as part of the [ErrorProne ](https://github.com/google/error-prone)static analysis tool. This annotation is a way for API owners to declare, “this type should not be mocked because better alternatives exist.”

在Google，我們已經看到了足夠多的過度依賴模擬框架的測試，這促使我們在Java中建立了@DoNotMock註解，它可以作為[ErrorProne](https://github.com/google/error-prone)靜態分析工具的一部分。這個註解是API所有者宣告的一種方式，"這個型別不應該被模擬，因為存在更好的替代方案"。

If an engineer attempts to use a mocking framework to create an instance of a class or interface that has been annotated as @DoNotMock, as demonstrated in [Example 13-10](#_bookmark1112), they will see an error directing them to use a more suitable test strategy, such as a real implementation or a fake. This annotation is most commonly used for value objects that are simple enough to use as-is, as well as for APIs that have well-engineered fakes available.

如果工程師試圖使用模擬框架來建立一個被註解為@DoNotMock的類別或介面的實例，如例13-10所示，他們會看到一個錯誤，指示他們使用更合適的測試策略，如真實的實現或偽造。這個註解最常用於那些簡單到可以按原樣使用的值物件，以及那些有精心設計的偽造的API。

*Example* *13-10. The @DoNotMock annotation*

```java
@DoNotMock("Use SimpleQuery.create() instead of mocking.")
public abstract class Query {
  public abstract String getQueryValue();
}
```

Why would an API owner care? In short, it severely constrains the API owner’s ability to make changes to their implementation over time. As we’ll explore later in the chapter, every time a mocking framework is used for stubbing or interaction testing, it duplicates behavior provided by the API.

為什麼API所有者會在意這個問題呢？簡而言之，它嚴重限制了API所有者隨時間對其實現進行更改的能力。正如我們在本章後面將探討的那樣，每次使用模擬框架進行存打樁或互動測試時，它都會複製API提供的行為。

When the API owner wants to change their API, they might find that it has been mocked thousands or even tens of thousands of times throughout Google’s codebase! These test doubles are very likely to exhibit behavior that violates the API contract of the type being mocked—for instance, returning null for a method that can never return null. Had the tests used the real implementation or a fake, the API owner could make changes to their implementation without first fixing thousands of flawed tests.

當API所有者想要改變他們的API時，他們可能會發現它已經在整個Google的程式碼函式庫中被模擬了數千次甚至上萬次！這些測試替代很可能表現出違反被模擬型別的API契約的行為——例如，為一個永遠不能返回null的方法返回null。如果測試使用的是真正的實現或偽造，API所有者可以對他們的實現進行修改，而不需要先修復成千上萬的有缺陷的測試。

-----

### How to Decide When to Use a Real Implementation 如何決定何時使用真實實現

A real implementation is preferred if it is fast, deterministic, and has simple dependencies. For example, a real implementation should be used for a [*value object*](https://oreil.ly/UZiXP). Examples include an amount of money, a date, a geographical address, or a collection class such as a list or a map.

如果真實實現速度快、確定性強且依賴性簡單，則首選真實實現。例如，一個真實實現應該被用於[*值物件*](https://oreil.ly/UZiXP)。例子包括一筆錢、一個日期、一個地理位置，或者一個集合類別，如列表或地圖。

However, for more complex code, using a real implementation often isn’t feasible. There might not be an exact answer on when to use a real implementation or a test double given that there are trade-offs to be made, so you need to take the following considerations into account.

然而，對於更復雜的程式碼，使用真實實現通常是不可行的。考慮到需要進行權衡，可能沒有關於何時使用真實實現或測試替代的確切答案，因此需要考慮以下因素。

#### Execution time 執行時間

One of the most important qualities of unit tests is that they should be fast—you want to be able to continually run them during development so that you can get quick feedback on whether your code is working (and you also want them to finish quickly when run in a CI system). As a result, a test double can be very useful when the real implementation is slow.

單元測試的一個最重要的特性是它們應該是快速的——你希望能夠在開發過程中持續執行它們，以便能夠快速獲得程式碼是否正常工作的反饋（你還希望它們在CI系統中執行時能夠快速完成）因此，當實際實現緩慢時，測試替代可能非常有用。

How slow is too slow for a unit test? If a real implementation added one millisecond to the running time of each individual test case, few people would classify it as slow. But what if it added 10 milliseconds, 100 milliseconds, 1 second, and so on?

對於一個單元測試來說，多慢才算慢？如果一個真正實現在每個單獨的測試用例的執行時間上增加一毫秒，很少有人會將其歸類別為慢。但如果它增加了10毫秒，100毫秒，1秒等等呢？

There is no exact answer here—it can depend on whether engineers feel a loss in productivity, and how many tests are using the real implementation (one second extra per test case may be reasonable if there are five test cases, but not if there are 500). For borderline situations, it is often simpler to use a real implementation until it becomes too slow to use, at which point the tests can be updated to use a test double instead.

這裡沒有確切的答案——它可能取決於工程師是否感到生產力下降，以及有多少測試正在使用實際實現（如果有5個測試用例，每個測試用例多一秒鐘可能是合理的，但如果有500個測試用例就不一樣了）。對於臨界情況，通常更容易使用實際實現，直到它變得太慢而無法使用，此時可以更新測試以使用測試替代。

Parellelization of tests can also help reduce execution time. At Google, our test infrastructure makes it trivial to split up tests in a test suite to be executed across multiple servers. This increases the cost of CPU time, but it can provide a large savings in developer time. We discuss this more in [Chapter 18](#_bookmark1596).

測試的並行化也有助於減少執行時間。在谷歌，我們的測試基礎設施使得將測試套件中的測試拆分到多個伺服器上執行變得非常簡單。這增加了CPU的成本，但它可以為開發人員節省大量時間。我們在第18章中對此有更多的討論。

Another trade-off to be aware of: using a real implementation can result in increased build times given that the tests need to build the real implementation as well as all of its dependencies. Using a highly scalable build system like [Bazel ](https://bazel.build/)can help because it caches unchanged build artifacts.

另一個需要注意的權衡：使用一個真實實現會導致建構時間的增加，因為測試需要建構真實實現以及它的所有依賴。使用像[Bazel](https://bazel.build/)這樣的高度可擴充套件的建構系統會有幫助，因為它快取了未改變的建構構件。

#### Determinism 確定性

A test is [*deterministic* ](https://oreil.ly/brxJl)if, for a given version of the system under test, running the test always results in the same outcome; that is, the test either always passes or always fails. In contrast, a test is [*nondeterministic* ](https://oreil.ly/5pG0f)if its outcome can change, even if the system under test remains unchanged.

如果對於被測系統的給定版本，執行測試的結果總是相同的，也就是說，測試要麼總是透過，要麼總是失敗，那麼這個測試就是[*確定性*](https://oreil.ly/brxJl)。相反，如果一個測試的結果可以改變，即使被測系統保持不變，那麼它就是[*非確定性*](https://oreil.ly/5pG0f)。

[Nondeterminism in tests ](https://oreil.ly/71OFU)can lead to flakiness—tests can occasionally fail even when there are no changes to the system under test. As discussed in [Chapter 11](#_bookmark838), flakiness harms the health of a test suite if developers start to distrust the results of the test and ignore failures. If use of a real implementation rarely causes flakiness, it might not warrant a response, because there is little disruption to engineers. But if flakiness hap‐pens often, it might be time to replace a real implementation with a test double because doing so will improve the fidelity of the test.

[測試中的非確定性](https://oreil.ly/71OFU)會導致鬆散性——即使被測系統沒有變化，測試也會偶爾失敗。正如在第11章中所討論的，如果開發人員開始不相信測試的結果並忽視失敗，那麼鬆散性會損害測試套件的健康。如果使用一個真正實現很少引起鬆散性，它可能不需要響應失敗，因為對工程師的干擾很小。但是，如果經常發生故障，可能是時候用一個測試替代真實實現了，因為這樣做會提高測試的模擬度。

A real implementation can be much more complex compared to a test double, which increases the likelihood that it will be nondeterministic. For example, a real implementation that utilizes multithreading might occasionally cause a test to fail if the output of the system under test differs depending on the order in which the threads are executed.

與測試替代相比，真正實現可能要複雜得多，這增加了它不確定性的概率。例如，如果被測系統的輸出因執行緒的執行順序不同而不同，利用多執行緒的真實實現可能偶爾會導致測試失敗。

A common cause of nondeterminism is code that is not [hermetic](https://oreil.ly/aes__); that is, it has dependencies on external services that are outside the control of a test. For example, a test that tries to read the contents of a web page from an HTTP server might fail if the server is overloaded or if the web page contents change. Instead, a test double should be used to prevent the test from depending on an external server. If using a test double is not feasible, another option is to use a hermetic instance of a server, which has its life cycle controlled by the test. Hermetic instances are discussed in more detail in the next chapter.

不確定性的一個常見原因是程式碼不夠封閉；也就是說，它依賴於測試無法控制的外部服務。例如，如果伺服器過載或網頁內容更改，嘗試從HTTP伺服器讀取網頁內容的測試可能會失敗。相反，應該使用測試替代來防止測試依賴於外部伺服器。如果使用測試工具不可行，另一種選擇是使用伺服器的封閉實例，其生命週期由測試控制。下一章將更詳細地討論封閉實例。

Another example of nondeterminism is code that relies on the system clock given that the output of the system under test can differ depending on the current time. Instead of relying on the system clock, a test can use a test double that hardcodes a specific time.

不確定性的另一個例子是依賴於系統時鐘的程式碼，因為被測系統的輸出可能因當前時間而異。測試可以使用硬編碼特定時間的測試替代，而不是依賴於系統時鐘。

#### Dependency construction 依賴關係的建構

When using a real implementation, you need to construct all of its dependencies. For example, an object needs its entire dependency tree to be constructed: all objects that it depends on, all objects that these dependent objects depend on, and so on. A test double often has no dependencies, so constructing a test double can be much simpler compared to constructing a real implementation.

當使用真實實現時，你需要構造它的所有依賴項。例如，一個物件需要構造其整個依賴關係樹：它所依賴的所有物件，這些依賴物件所依賴的所有物件，等等。測試替代通常沒有依賴項，因此與建構實際實現相比，建構測試替代要簡單得多。

As an extreme example, imagine trying to create the object in the code snippet that follows in a test. It would be time consuming to determine how to construct each individual object. Tests will also require constant maintenance because they need to be updated when the signature of these objects’ constructors is modified:

作為一個極端的例子，想象一下嘗試在測試中後面的程式碼段中建立物件。確定如何構造每個單獨的物件將非常耗時。測試還需要持續維護，因為當這些物件的建構函式的簽名被修改時，測試需要更新：

```jav
Foo foo = new Foo(new A(new B(new C()), new D()), new E(), ..., new Z());
```

It can be tempting to instead use a test double because constructing one can be trivial. For example, this is all it takes to construct a test double when using the Mockito mocking framework:

使用測試替代是很有誘惑力的，因為建構一個測試替代是很簡單的。例如，在使用模擬框架時，這就是建構一個測試替代的全部內容：

```java
@Mock 
Foo mockFoo;
```

Although creating this test double is much simpler, there are significant benefits to using the real implementation, as discussed earlier in this section. There are also often significant downsides to overusing test doubles in this way, which we look at later in this chapter. So, a trade-off needs to be made when considering whether to use a real implementation or a test double.

儘管建立這個測試替代要簡單得多，但使用真正實現有很大的好處，正如本節前面所討論的。以這種方式過度使用測試替代往往也有很大的弊端，我們在本章後面會看一下。所以，在考慮是使用真實實現還是測試替身時，需要做一個權衡。

Rather than manually constructing the object in tests, the ideal solution is to use the same object construction code that is used in the production code, such as a factory method or automated dependency injection. To support the use case for tests, the object construction code needs to be flexible enough to be able to use test doubles rather than hardcoding the implementations that will be used for production.

與其在測試中手動建構物件，理想的解決方案是使用生產程式碼中使用的相同的物件建構程式碼，如工廠方法或自動依賴注入。為了支援測試的使用情況，物件構造程式碼需要有足夠的靈活性，能夠使用測試替代，而不是硬編碼將用於生產的實現。

## Faking 偽造測試

If using a real implementation is not feasible within a test, the best option is often to use a fake in its place. A fake is preferred over other test double techniques because it behaves similarly to the real implementation: the system under test shouldn’t even be able to tell whether it is interacting with a real implementation or a fake. [Example 13-11 ](#_bookmark1127)illustrates a fake file system. 

如果在測試中使用真實實現是不可行的，那麼最好的選擇通常是使用偽造實現。與其他測試替代技術相比，偽造測試技術更受歡迎，因為它的行為類似於真實實現：被測試的系統甚至不能判斷它是與真實實現互動還是與偽造實現互動。示例13-11示範了一個偽造檔案系統。

*Example* *13-11.* *A* *fake* *file* *system*

```java
// This fake implements the FileSystem interface. This interface is also
// used by the real implementation.
public class FakeFileSystem implements FileSystem {
    // Stores a map of file name to file contents. The files are stored in
    // memory instead of on disk since tests shouldn’t need to do disk I/O.
    private Map < String, String > files = new HashMap< > ();
    
    @Override
    public void writeFile(String fileName, String contents) {
        // Add the file name and contents to the map.
        files.add(fileName, contents);
    }
  
    @Override
    public String readFile(String fileName) {
        String contents = files.get(fileName);
        // The real implementation will throw this exception if the
        // file isn’t found, so the fake must throw it too.
        if(contents == null) {
            throw new FileNotFoundException(fileName);
        }
        return contents;
    }
}
```

### Why Are Fakes Important? 為什麼偽造測試很重要？

Fakes can be a powerful tool for testing: they execute quickly and allow you to effectively test your code without the drawbacks of using real implementations.

偽造測試是一個強大的測試工具：它們可以快速執行，並允許你有效地測試程式碼，而沒有使用真實實現的缺點。

A single fake has the power to radically improve the testing experience of an API. If you scale that to a large number of fakes for all sorts of APIs, fakes can provide an enormous boost to engineering velocity across a software organization.

一個偽造的API就可以從根本上改善API的測試體驗。如果將其擴充套件到各種API的大量偽造，偽造可以極大地提高整個軟體組織的工程速度。

At the other end of the spectrum, in a software organization where fakes are rare, velocity will be slower because engineers can end up struggling with using real implementations that lead to slow and flaky tests. Or engineers might resort to other test double techniques such as stubbing or interaction testing, which, as we’ll examine later in this chapter, can result in tests that are unclear, brittle, and less effective.

另一方面，在一個使用偽造測試很少的軟體組織中，速度會慢一些，因為工程師最終會在使用真正實現時遇到困難，從而導致測試緩慢和不穩定。或者工程師可能會求助於其他測試替代技術，如打樁或互動測試，正如我們將在本章後面討論的那樣，這些技術可能會導致測試不清晰、脆弱且效率較低。

### When Should Fakes Be Written? 什麼時候應該寫偽造測試？

A fake requires more effort and more domain experience to create because it needs to behave similarly to the real implementation. A fake also requires maintenance: whenever the behavior of the real implementation changes, the fake must also be updated to match this behavior. Because of this, the team that owns the real implementation should write and maintain a fake.

偽造測試需要更多的努力和更多的領域經驗來建立，因為它需要與真實實現類似的行為。偽造實現程式碼還需要維護：當真實實現的行為發生更改時，偽造實現程式碼也必須更新以匹配此行為。因此，擁有真實實現的團隊應該編寫並維護一個偽造實現程式碼。

If a team is considering writing a fake, a trade-off needs to be made on whether the productivity improvements that will result from the use of the fake outweigh the costs of writing and maintaining it. If there are only a handful of users, it might not be worth their time, whereas if there are hundreds of users, it can result in an obvious productivity improvement.

如果一個團隊正在考慮編寫一個偽造測試，就需要權衡使用偽造測試所帶來的生產力的提高是否超過了編寫和維護的成本。如果只有少數幾個使用者，可能不值得他們花費時間，而如果有幾百個使用者，這可以顯著提高生產力。

To reduce the number of fakes that need to be maintained, a fake should typically be created only at the root of the code that isn’t feasible for use in tests. For example, if a database can’t be used in tests, a fake should exist for the database API itself rather than for each class that calls the database API.

為了減少需要維護的偽造測試程式碼的數量，偽造測試程式碼通常應該只在測試中不可行的程式碼根處建立。例如，如果一個數據函式庫不能在測試中使用，那麼應該為資料庫API本身編寫一個偽造測試，而不是為呼叫資料庫API的每個類別編寫。

Maintaining a fake can be burdensome if its implementation needs to be duplicated across programming languages, such as for a service that has client libraries that allow the service to be invoked from different languages. One solution for this case is to create a single fake service implementation and have tests configure the client libraries to send requests to this fake service. This approach is more heavyweight compared to having the fake written entirely in memory because it requires the test to communicate across processes. However, it can be a reasonable trade-off to make, as long as the tests can still execute quickly.

如果需要跨程式語言複製偽造實現程式碼的實現，例如對於具有允許從不同語言呼叫服務的客戶端函式庫的服務，則維護偽造實現程式碼可能會很麻煩。這種情況下的一個解決方案是建立一個偽造服務實現，並讓測試配置客戶端函式庫以向該偽造服務傳送請求。與將偽造實現程式碼完全寫入記憶體相比，這種方法更為重要，因為它需要測試跨程序進行通訊。但是，只要測試仍然可以快速執行，那麼這是一個合理的權衡。

### The Fidelity of Fakes 偽造測試的模擬度

Perhaps the most important concept surrounding the creation of fakes is *fidelity*; in other words, how closely the behavior of a fake matches the behavior of the real implementation. If the behavior of a fake doesn’t match the behavior of the real implementation, a test using that fake is not useful—a test might pass when the fake is used, but this same code path might not work properly in the real implementation.

也許圍繞著建立偽造測試的最重要的概念是*模擬度*；換句話說，偽造測試的行為與真實實現的行為的匹配程度。如果偽造測試的行為與真實實現的行為不匹配，那麼使用該偽造測試就沒有用處——當使用該偽造測試時，測試可能會透過，但同樣的程式碼路徑在真實實現中可能無法正常工作。

Perfect fidelity is not always feasible. After all, the fake was necessary because the real implementation wasn’t suitable in one way or another. For example, a fake database would usually not have fidelity to a real database in terms of hard drive storage because the fake would store everything in memory.

完美的模擬並不總是可行的。畢竟，偽造是必要的，因為真實實現在某種程度上並不適合。例如，在硬碟儲存方面，一個偽造資料庫通常不會與真正的資料庫一樣，因為偽造資料庫會把所有東西都儲存在記憶體中。

Primarily, however, a fake should maintain fidelity to the API contracts of the real implementation. For any given input to an API, a fake should return the same output and perform the same state changes of its corresponding real implementation. For example, for a real implementation of database.save(itemId), if an item is successfully saved when its ID does not yet exist but an error is produced when the ID already exists, the fake must conform to this same behavior.

然而，主要的是，偽造測試應該保持對真實實現的API契約的完整性。對於API的任何給定的輸入，偽造測試應該返回相同的輸出，並對其相應的實際實現執行相同的狀態更改。例如，對於資料庫.save(itemId)的真實實現，如果一個專案在其ID不存在的情況下被成功儲存，但在ID已經存在的情況下會產生一個錯誤，偽造資料庫必須符合這個相同的行為。

One way to think about this is that the fake must have perfect fidelity to the real implementation, but *only from the perspective of the test*. For example, a fake for a hashing API doesn’t need to guarantee that the hash value for a given input is exactly the same as the hash value that is generated by the real implementation—tests likely don’t care about the specific hash value, only that the hash value is unique for a given input. If the contract of the hashing API doesn’t make guarantees of what specific hash values will be returned, the fake is still conforming to the contract even if it doesn’t have perfect fidelity to the real implementation.

一種思考方式是，偽造測試必須對真正的實現有完美的模擬度，但只能從測試的角度來看。例如，一個偽造hash API不需要保證給定輸入的hash值與真實實現產生的hash值完全相同——測試可能不關心具體的hash值，只關心給定輸入的hash值是唯一的。如果hash API的契約沒有保證將返回哪些特定的hash值，那麼偽造函式仍然符合契約，即使它與真實實現沒有完美的模擬度。

Other examples where perfect fidelity typically might not be useful for fakes include latency and resource consumption. However, a fake cannot be used if you need to explicitly test for these constraints (e.g., a performance test that verifies the latency of a function call), so you would need to resort to other mechanisms, such as by using a real implementation instead of a fake.

完美的模擬度通常不適用於偽造的其他示例包括延遲和資源消耗。但是，如果你需要顯式測試這些約束（例如，驗證函式呼叫延遲的效能測試），則不能使用偽造函式，因此你需要求助於其他機制，例如使用真實實現而不是偽造函式。

A fake might not need to have 100% of the functionality of its corresponding real implementation, especially if such behavior is not needed by most tests (e.g., error handling code for rare edge cases). It is best to have the fake fail fast in this case; for example, raise an error if an unsupported code path is executed. This failure communicates to the engineer that the fake is not appropriate in this situation.

偽造實現程式碼可能不需要擁有其對應的真實實現的100%功能，尤其是在大多數測試不需要這種行為的情況下（例如，罕見邊緣情況下的錯誤處理程式碼）。在這種情況下，最好讓偽造測試快速失效；例如，如果執行了不受支援的程式碼路徑，則引發錯誤。該故障告知工程師，在這種情況下，偽造測試是不合適的。

### Fakes Should Be Tested  偽造測試應當被測試

A fake must have its *own* tests to ensure that it conforms to the API of its corresponding real implementation. A fake without tests might initially provide realistic behavior, but without tests, this behavior can diverge over time as the real implementation evolves.

偽造測試必須有自己的*測試*，以確保它符合其相應的真實實現的API。沒有測試的偽造最初可能會提供真實的行為，但如果沒有測試，隨著時間的推移，這種行為會隨著真實實現的發展而發生變化。

One approach to writing tests for fakes involves writing tests against the API’s public interface and running those tests against both the real implementation and the fake (these are known as [*contract tests*](https://oreil.ly/yuVlX)). The tests that run against the real implementation will likely be slower, but their downside is minimized because they need to be run only by the owners of the fake.

為偽造測試編寫測試的一種方法是針對API的公共介面編寫測試，並針對真實實現和偽造測試執行這些測試（這些被稱為[*合同測試*](https://oreil.ly/yuVlX)）。針對真實實現執行的測試可能會更慢，但它們的缺點會被最小化，因為它們只需要由偽造實現程式碼的所有者執行。

### What to Do If a Fake Is Not Available 如果沒有偽造測試怎麼辦？

If a fake is not available, first ask the owners of the API to create one. The owners might not be familiar with the concept of fakes, or they might not realize the benefit they provide to users of an API.

如果沒有偽造測試，首先要求API的所有者建立一個。所有者可能不熟悉偽造測試的概念，或者他們可能沒有意識到偽造測試對API使用者的好處。

If the owners of an API are unwilling or unable to create a fake, you might be able to write your own. One way to do this is to wrap all calls to the API in a single class and then create a fake version of the class that doesn’t talk to the API. Doing this can also be much simpler than creating a fake for the entire API because often you’ll need to use only a subset of the API’s behavior anyway. At Google, some teams have even contributed their fake to the owners of the API, which has allowed other teams to benefit from the fake.

如果一個API的所有者不願意或無法建立一個偽造測試，你可以寫一個。實現這一點的一種方法是將對API的所有呼叫封裝在一個類別中，然後建立一個不與API對話的類別的偽造測試版本。這樣做也比為整個API建立一個偽造測試API簡單得多，因為通常你只需要使用API行為的一個子集。在谷歌，一些團隊甚至將他們的偽造測試貢獻給API的所有者，這使得其他團隊可以從偽造測試中獲益。

Finally, you could decide to settle on using a real implementation (and deal with the trade-offs of real implementations that are mentioned earlier in this chapter), or resort to other test double techniques (and deal with the trade-offs that we will mention later in this chapter).

最後，你可以決定定位於使用真實實現（並處理本章前面提到的真實實現的權衡問題），或者求助於其他測試替代技術（並處理本章後面提到的權衡問題）。

In some cases, you can think of a fake as an optimization: if tests are too slow using a real implementation, you can create a fake to make them run faster. But if the speedup from a fake doesn’t outweigh the work it would take to create and maintain the fake, it would be better to stick with using the real implementation.

在某些情況下，可以將偽造實現程式碼視為優化：如果使用真實實現的測試太慢，可以建立虛擬碼以使它們執行得更快。但是，如果偽造實現程式碼的加速比不超過建立和維護偽造實現程式碼所需的工作量，那麼最好還是堅持使用真實實現。

## Stubbing 打樁

As discussed earlier in this chapter, stubbing is a way for a test to hardcode behavior for a function that otherwise has no behavior on its own. It is often a quick and easy way to replace a real implementation in a test. For example, the code in [Example 13-12 ](#_bookmark1144)uses stubbing to simulate the response from a credit card server.

正如本章前面所討論的，打樁是一種測試函式硬編碼行為的方法，否則函式本身就沒有行為。它通常是一種快速而簡單的方法來替代測試中的真實實現。例如，例13-12中的程式碼使用打樁來模擬信用卡伺服器的響應。

*Example* *13-12.* *Using* *stubbing* *to* *simulate* *responses*

```java
@Test public void getTransactionCount() {
transactionCounter = new TransactionCounter(mockCreditCardServer);
// Use stubbing to return three transactions.
when(mockCreditCardServer.getTransactions()).thenReturn( newList(TRANSACTION_1, TRANSACTION_2, TRANSACTION_3));
assertThat(transactionCounter.getTransactionCount()).isEqualTo(3);
}
```

### The Dangers of Overusing Stubbing  過度使用打樁的危害

Because stubbing is so easy to apply in tests, it can be tempting to use this technique anytime it’s not trivial to use a real implementation. However, overuse of stubbing can result in major losses in productivity for engineers who need to maintain these tests.

因為打樁在測試中很容易應用，所以在使用真實實現不容易的情況下，使用這種技術是很誘惑力的。然而，過度使用打樁會導致需要維護這些測試的工程師的生產力的重大損失。

#### Tests become unclear 測試變得不清晰

Stubbing involves writing extra code to define the behavior of the functions being stubbed. Having this extra code detracts from the intent of the test, and this code can be difficult to understand if you’re not familiar with the implementation of the system under test.

打樁涉及編寫額外的程式碼來定義被打樁的函式的行為。額外的程式碼會影響測試的意圖，如果你不熟悉被測系統的實現，這些程式碼會很難理解。

A key sign that stubbing isn’t appropriate for a test is if you find yourself mentally stepping through the system under test in order to understand why certain functions in the test are stubbed.

打樁不適用於測試的一個關鍵標誌是，如果你發現自己為了理解為什麼測試中的某些功能是打樁的，而在思路已經躍出了被測系統。

#### Tests become brittle 測試變得脆弱

Stubbing leaks implementation details of your code into your test. When implementation details in your production code change, you’ll need to update your tests to reflect these changes. Ideally, a good test should need to change only if user-facing behavior of an API changes; it should remain unaffected by changes to the API’s implementation.

打樁測試將你的程式碼的實現細節洩露給你的測試。當生產程式碼中的實現細節改變時，你需要更新你的測試以反映這些變化。理想情況下，一個好的測試應該只在API面向使用者的行為發生變化時才需要改變；它應該不受API實現變化的影響。

#### Tests become less effective 測試有效性降低

With stubbing, there is no way to ensure the function being stubbed behaves like the real implementation, such as in a statement like that shown in the following snippet that hardcodes part of the contract of the add() method (*“If 1 and 2 are passed in, 3* *will be returned”*):

在打樁的情況下，沒有辦法確保被打樁的函式表現得像真實實現，比如像下面這個片段中的語句，硬編碼了add()方法的部分契約（*"如果傳入1和2，3將被返回 "*）。

```java
when(stubCalculator.add(1, 2)).thenReturn(3);
```

Stubbing is a poor choice if the system under test depends on the real implementation’s contract because you will be forced to duplicate the details of the contract, and there is no way to guarantee that the contract is correct (i.e., that the stubbed function has fidelity to the real implementation).

如果被測試的系統依賴於真實實現的契約，打樁測試是一個糟糕的選擇，因為你將被迫複製契約的細節，而且沒有辦法保證契約的正確性（即，打樁函式對真實實現的模擬度）。

Additionally, with stubbing there is no way to store state, which can make it difficult to test certain aspects of your code. For example, if you call database.save(item) on either a real implementation or a fake, you might be able to retrieve the item by calling database.get(item.id()) given that both of these calls are accessing internal state, but with stubbing, there is no way to do this.

此外，使用打樁測試無法儲存狀態，這會使測試程式碼的某些方面變得困難。例如，如果你在一個真實實現或位置實現上呼叫database.save(item)，你可能會透過呼叫database.get(item.id())來檢索專案，因為這兩個呼叫都是在訪問內部狀態，但在打樁測試中，沒有辦法這樣做。

An example of overusing stubbing.

一個過度使用打樁測試的例子。

[Example 13-13 ](#_bookmark1151)illustrates a test that overuses stubbing.

例13-13說明了一個過度使用打樁的測試。

*Example* *13-13.* *Overuse* *of* *stubbing*

```java
@Test
public void creditCardIsCharged() {
    // Pass in test doubles that were created by a mocking framework.
    paymentProcessor = new PaymentProcessor(mockCreditCardServer, mockTransactionProcessor);
    // Set up stubbing for these test doubles. 
    when(mockCreditCardServer.isServerAvailable()).thenReturn(true);
    when(mockTransactionProcessor.beginTransaction()).thenReturn(transaction);
    when(mockCreditCardServer.initTransaction(transaction)).thenReturn(true);
    when(mockCreditCardServer.pay(transaction, creditCard, 500)).thenReturn(false);
    when(mockTransactionProcessor.endTransaction()).thenReturn(true);
    // Call the system under test.
    paymentProcessor.processPayment(creditCard, Money.dollars(500));
    // There is no way to tell if the pay() method actually carried out the
    // transaction, so the only thing the test can do is verify that the
    // pay() method was called.
    verify(mockCreditCardServer).pay(transaction, creditCard, 500);
}
```

[Example 13-14 ](#_bookmark1153)rewrites the same test but avoids using stubbing. Notice how the test is shorter and that implementation details (such as how the transaction processor is used) are not exposed in the test. No special setup is needed because the credit card server knows how to behave.

例13-14重寫了同樣的測試，但避免了使用打樁測試方式。注意這個測試是如何精簡的，並且在測試中沒有暴露實現細節（比如如何使用交易處理器）。不需要特別的設定，因為信用卡伺服器知道如何操作。

*Example* *13-14.* *Refactoring* *a* *test* *to* *avoid* *stubbing*

```java
@Test
public void creditCardIsCharged() {
    paymentProcessor = new PaymentProcessor(creditCardServer, transactionProcessor);
    // Call the system under test.
    paymentProcessor.processPayment(creditCard, Money.dollars(500));
    // Query the credit card server state to see if the payment went through.
    assertThat(creditCardServer.getMostRecentCharge(creditCard)).isEqualTo(500);
}
```

We obviously don’t want such a test to talk to an external credit card server, so a fake credit card server would be more suitable. If a fake isn’t available, another option is to use a real implementation that talks to a hermetic credit card server, although this will increase the execution time of the tests. (We explore hermetic servers in the next chapter.)

顯然，我們不希望這樣的測試與外部信用卡伺服器互動，因此更適合使用假信用卡伺服器。如果一個偽造不可用，另一個選擇是使用一個真實實現，與一個封閉的信用卡伺服器互動，儘管這會增加測試的執行時間。（我們將在下一章中探討封閉伺服器。）

### When Is Stubbing Appropriate? 什麼情況下才適合使用打樁測試？

Rather than a catch-all replacement for a real implementation, stubbing is appropriate when you need a function to return a specific value to get the system under test into a certain state, such as [Example 13-12](#_bookmark1144) that requires the system under test to return a non-empty list of transactions. Because a function’s behavior is defined inline in the test, stubbing can simulate a wide variety of return values or errors that might not be possible to trigger from a real implementation or a fake.

當你需要一個函式返回一個特定的值以使被測系統進入某種狀態時，打樁方式就很合適，而不是真實實現的萬能替代品，例如例13-12要求被測系統返回一個非空的事務列表。因為一個函式的行為是在測試中內聯定義的，所以打樁可以模擬各種各樣的返回值或錯誤，而這些返回值或錯誤可能無法從真實實現或偽造測試中觸發。

To ensure its purpose is clear, each stubbed function should have a direct relationship with the test’s assertions. As a result, a test typically should stub out a small number of functions because stubbing out many functions can lead to tests that are less clear. A test that requires many functions to be stubbed can be a sign that stubbing is being overused, or that the system under test is too complex and should be refactored.

為了確保其目的明確，每個打樁函式應該與測試的斷言直接相關。因此，一個測試通常應該打樁少量的函式，因為打樁太多會導致函式不夠清晰。一個需要打樁許多函式的測試是一個跡象，表明打樁被過度使用，或者被測系統過於複雜，應該被重構。

Note that even when stubbing is appropriate, real implementations or fakes are still preferred because they don’t expose implementation details and they give you more guarantees about the correctness of the code compared to stubbing. But stubbing can be a reasonable technique to use, as long as its usage is constrained so that tests don’t become overly complex.

請注意，即使打樁測試是合適的，真實實現或偽造測試仍然是首選，因為它們不會暴露實現的細節，與打樁測試相比，它們能給你更多關於程式碼的正確性的保證。但打樁可以是一種合理的技術，只要它的使用受到限制，使測試不會變得過於複雜。

## Interaction Testing  互動測試

As discussed earlier in this chapter, interaction testing is a way to validate how a function is called without actually calling the implementation of the function.

正如本章前面所討論的，互動測試是一種驗證函式如何被呼叫的方法，而不需要實際呼叫該函式的實現。

Mocking frameworks make it easy to perform interaction testing. However, to keep tests useful, readable, and resilient to change, it’s important to perform interaction testing only when necessary.

模擬框架使執行互動測試變得容易。然而，為了保持測試的有用性、可讀性和應變能力，只在必要時執行互動測試是很重要的。

### Prefer State Testing Over Interaction Testing 推薦狀態測試而非互動測試

In contrast to interaction testing, it is preferred to test code through [*state* *testing*](https://oreil.ly/k3hSR).

與互動測試相比，最好是透過[*狀態測試*](https://oreil.ly/k3hSR)來測試程式碼。

With state testing, you call the system under test and validate that either the correct value was returned or that some other state in the system under test was properly changed. [Example 13-15 ](#_bookmark1162)presents an example of state testing.

透過狀態測試，你可以呼叫被測系統，並驗證返回的值是否正確，或者被測系統中的其他狀態是否已正確更改。示例13-15給出了一個狀態測試示例。

*Example 13-15. State testing*

 ```java
 @Test
 public void sortNumbers() {
     NumberSorter numberSorter = new NumberSorter(quicksort, bubbleSort);
     // Call the system under test.
     List sortedList = numberSorter.sortNumbers(newList(3, 1, 2));
     // Validate that the returned list is sorted. It doesn’t matter which
     // sorting algorithm is used, as long as the right result was returned.
     assertThat(sortedList).isEqualTo(newList(1, 2, 3));
 }
 ```

[Example 13-16 ](#_bookmark1163)illustrates a similar test scenario but instead uses interaction testing. Note how it’s impossible for this test to determine that the numbers are actually sorted, because the test doubles don’t know how to sort the numbers—all it can tell you is that the system under test tried to sort the numbers.

 示例13-16說明了一個類似的測試場景，但使用了互動測試。請注意，此測試無法確定數字是否實際已排序，因為測試替代不知道如何對數字進行排序——它所能告訴你的是，被測試系統嘗試對數字進行排序。

*Example* *13-16.* *Interaction* *testing*

```java
@Test 
public void sortNumbers_quicksortIsUsed() {
    // Pass in test doubles that were created by a mocking framework.
    NumberSorter numberSorter = new NumberSorter(mockQuicksort, mockBubbleSort);
    // Call the system under test.
    numberSorter.sortNumbers(newList(3, 1, 2));
    // Validate that numberSorter.sortNumbers() used quicksort. The test
    // will fail if mockQuicksort.sort() is never called (e.g., if
    // mockBubbleSort is used) or if it’s called with the wrong arguments.
    verify(mockQuicksort).sort(newList(3, 1, 2));
}
```

At Google, we’ve found that emphasizing state testing is more scalable; it reduces test brittleness, making it easier to change and maintain code over time.

在谷歌，我們發現強調狀態測試更具可擴充性；它降低了測試的脆弱性，使得隨著時間的推移更容易變更和維護程式碼。

The primary issue with interaction testing is that it can’t tell you that the system under test is working properly; it can only validate that certain functions are called as expected. It requires you to make an assumption about the behavior of the code; for example, “*If* *database.save(item) is called, we assume the item will be saved to the database.*” State testing is preferred because it actually validates this assumption (such as by saving an item to a database and then querying the database to validate that the item exists).

互動測試的主要問題是它不能告訴你被測試的系統是否正常工作；它只能驗證是否按預期呼叫了某些函式。它要求你對程式碼的行為做出假設；例如，首選“如果”狀態測試，因為它實際上驗證了該假設（例如，將專案儲存到資料庫，然後查詢資料庫以驗證該專案是否存在）。如果呼叫了*database.save(item)*，則假定該項將儲存到資料庫中。

Another downside of interaction testing is that it utilizes implementation details of the system under test—to validate that a function was called, you are exposing to the test that the system under test calls this function. Similar to stubbing, this extra code makes tests brittle because it leaks implementation details of your production code into tests. Some people at Google jokingly refer to tests that overuse interaction testing as [*change-detector* *tests* ](https://oreil.ly/zkMDu)because they fail in response to any change to the production code, even if the behavior of the system under test remains unchanged.

互動測試的另一個缺點是，它利用被測系統的實現細節——驗證某個函式是否被呼叫，你向測試暴露了被測系統呼叫這個函式。與打樁類似，這個額外的程式碼使測試變得脆弱，因為它將生產程式碼的實現細節洩漏到測試中。谷歌的一些人開玩笑地把過度使用互動測試的測試稱為[*變更檢測器測試*](https://oreil.ly/zkMDu)，因為它們對生產程式碼的任何改變都會失敗，即使被測系統的行為保持不變。

### When Is Interaction Testing Appropriate? 什麼時候適合進行互動測試？

There are some cases for which interaction testing is warranted:  
- You cannot perform state testing because you are unable to use a real implementation or a fake (e.g., if the real implementation is too slow and no fake exists). As a fallback, you can perform interaction testing to validate that certain functions are called. Although not ideal, this does provide some basic level of confidence that the system under test is working as expected.
- Differences in the number or order of calls to a function would cause undesired behavior. Interaction testing is useful because it could be difficult to validate this behavior with state testing. For example, if you expect a caching feature to reduce the number of calls to a database, you can verify that the database object is not accessed more times than expected. Using Mockito, the code might look similar to this:

在某些情況下，互動測試是有必要的：  
- 你不能進行狀態測試，因為你無法使用真實實現或偽造實現（例如，如果真實實現太慢，而且沒有偽造測試存在）。作為備用方案，你可以進行互動測試以驗證某些函式被呼叫。雖然不是很理想，但這確實提供了一些基本的功能，即被測系統正在按照預期工作。
- 對一個函式的呼叫數量或順序的不同會導致不在預期內的行為。互動測試是有用的，因為用狀態測試可能很難驗證這種行為。例如，如果你期望一個快取功能能減少對資料庫的呼叫次數，你可以驗證資料庫物件的訪問次數沒有超過預期。使用Mockito，程式碼可能看起來類似於這樣：

```java
verify(databaseReader, atMostOnce()).selectRecords();
```

Interaction testing is not a complete replacement for state testing. If you are not able to perform state testing in a unit test, strongly consider supplementing your test suite with larger-scoped tests that do perform state testing. For instance, if you have a unit test that validates usage of a database through interaction testing, consider adding an integration test that can perform state testing against a real database. Larger-scope testing is an important strategy for risk mitigation, and we discuss it in the next chapter.

互動測試不能完全替代狀態測試。如果無法在單元測試中執行狀態測試，請強烈考慮用更大範圍的執行狀態測試的範圍測試來補充測試套件。例如，如果你有一個單元測試，透過互動測試來驗證資料庫的使用，考慮新增一個整合測試，可以對真實資料庫進行狀態測試。更大範圍的測試是減輕風險的重要策略，我們將在下一章中討論它。

#### Best Practices for Interaction Testing 互動測試的最佳實踐

When performing interaction testing, following these practices can reduce some of the impact of the aforementioned downsides.

在進行互動測試時，遵循這些做法可以減少上述弊端的一些影響。

#### Prefer to perform interaction testing only for state-changing functions 傾向於只對狀態改變的功能進行互動測試

When a system under test calls a function on a dependency, that call falls into one of two categories:
- *State-changing*
    Functions that have side effects on the world outside the system under test. Examples: 

```java
sendEmail(), saveRecord(), logAccess().
```

- *Non-state-changing*
Functions that don’t have side effects; they return information about the world outside the system under test and don’t modify anything. Examples: 
```java
getUser(), findResults(), readFile().
```

當被測系統呼叫一個依賴關係上的函式時，該呼叫屬於兩類別中的一類別：

*改變狀態*
	對被測系統以外的範圍有副作用的函式。例子：

```java
sendEmail(), saveRecord(), logAccess().
```

- *不改變狀態*
沒有副作用的函式；它們返回關於被測系統以外的範圍的資訊，不修改任何東西。例如：

```java
getUser(), findResults(), readFile()。
```

In general, you should perform interaction testing only for functions that are state- changing. Performing interaction testing for non-state-changing functions is usually redundant given that the system under test will use the return value of the function to do other work that you can assert. The interaction itself is not an important detail for correctness, because it has no side effects.

一般來說，你應該只對狀態變化的函式進行互動測試。考慮到被測系統將使用函式的返回值來執行你可以斷言的其他工作，對非狀態變化函式執行互動測試通常是多餘的。互動本身對於正確性來說不是一個重要的細節，因為它沒有副作用。

Performing interaction testing for non-state-changing functions makes your test brittle because you’ll need to update the test anytime the pattern of interactions changes. It also makes the test less readable given that the additional assertions make it more difficult to determine which assertions are important for ensuring correctness of the code. By contrast, state-changing interactions represent something useful that your code is doing to change state somewhere else.

對非狀態變化的函式進行互動測試會使你的測試變得很脆弱，因為你需要在互動模式發生變化時更新測試。由於附加的斷言使得確定哪些斷言對於確保程式碼的正確性很重要變得更加困難，因此它還使得測試的可讀性降低。相比之下，狀態改變的互動代表了你的程式碼為改變其他地方的狀態所做的有用的事情。

[Example 13-17](#_bookmark1171) demonstrates interaction testing on both state-changing and non- state-changing functions.

例13-17展示了對狀態變化和非狀態變化函式的互動測試。

*Example 13-17. State-changing and non-state-changing interactions*  *例13-17. 狀態改變和非狀態改變的相互作用*

```java
@Test
public void grantUserPermission() {
    UserAuthorizer userAuthorizer = new UserAuthorizer(mockUserService, mockPermissionDatabase);
    when(mockPermissionService.getPermission(FAKE_USER)).thenReturn(EMPTY);
    // Call the system under test.
    userAuthorizer.grantPermission(USER_ACCESS);
    // addPermission() is state-changing, so it is reasonable to perform
    // interaction testing to validate that it was called.
    verify(mockPermissionDatabase).addPermission(FAKE_USER, USER_ACCESS);
    // getPermission() is non-state-changing, so this line of code isn’t
    // needed. One clue that interaction testing may not be needed:
    // getPermission() was already stubbed earlier in this test.
    verify(mockPermissionDatabase).getPermission(FAKE_USER);
}
```

#### Avoid overspecification 避免過度規範化

In [Chapter 12](#_bookmark938), we discuss why it is useful to test behaviors rather than methods. This means that a test method should focus on verifying one behavior of a method or class rather than trying to verify multiple behaviors in a single test.

在第12章中，我們將討論為什麼測試行為比測試方法更有用。這意味著一個測試方法應該關注於驗證一個方法或類別的一個行為，而不是試圖在一個測試中驗證多個行為。

When performing interaction testing, we should aim to apply the same principle by avoiding overspecifying which functions and arguments are validated. This leads to tests that are clearer and more concise. It also leads to tests that are resilient to changes made to behaviors that are outside the scope of each test, so fewer tests will fail if a change is made to a way a function is called.

在進行互動測試時，我們應該透過避免過度指定哪些函式和引數被驗證，來達到應用同樣的原則。這將導致測試更清晰、更簡潔。這也導致了測試對每個測試範圍之外的行為的改變有彈性，所以如果改變了一個函式的呼叫方式，更少的測試會失敗。

[Example 13-18 ](#_bookmark1174)illustrates interaction testing with overspecification. The intention of the test is to validate that the user’s name is included in the greeting prompt, but the test will fail if unrelated behavior is changed.

示例13-18說明了過度規範的互動測試。測試的目的是驗證使用者名稱是否包含在問候語提示中，但如果不相關的行為發生更改，測試將失敗。

 *Example* *13-18.* *Overspecified* *interaction* *tests*

```java
@Test public void displayGreeting_renderUserName() {
    when(mockUserService.getUserName()).thenReturn("Fake User");
    userGreeter.displayGreeting();
    // Call the system under test.
    // The test will fail if any of the arguments to setText() are changed.
    verify(userPrompt).setText("Fake User", "Good morning!", "Version 2.1");
    // The test will fail if setIcon() is not called, even though this
    // behavior is incidental to the test since it is not related to
    // validating the user name.
    verify(userPrompt).setIcon(IMAGE_SUNSHINE);
}
```

[Example 13-19](#_bookmark1176) illustrates interaction testing with more care in specifying relevant arguments and functions. The behaviors being tested are split into separate tests, and each test validates the minimum amount necessary for ensuring the behavior it is testing is correct.

例13-19說明了互動測試在指定相關引數和函式時更加謹慎。被測試的行為被分成獨立的測試，每個測試都驗證了確保它所測試的行為是正確的所需的最小量。

 *Example 13-19. Well-specified interaction tests*  *例13-19.指向明確的互動檢驗*

```java
@Test 
public void displayGreeting_renderUserName() {
    when(mockUserService.getUserName()).thenReturn("Fake User");
    userGreeter.displayGreeting(); // Call the system under test. 
    verify(userPrompter).setText(eq("Fake User"), any(), any());
}

@Test 
public void displayGreeting_timeIsMorning_useMorningSettings() {
    setTimeOfDay(TIME_MORNING);
    userGreeter.displayGreeting(); // Call the system under test. 
    verify(userPrompt).setText(any(), eq("Good morning!"), any());
    verify(userPrompt).setIcon(IMAGE_SUNSHINE);
}
```

## Conclusion 總結

We’ve learned that test doubles are crucial to engineering velocity because they can help comprehensively test your code and ensure that your tests run fast. On the other hand, misusing them can be a major drain on productivity because they can lead to tests that are unclear, brittle, and less effective. This is why it’s important for engineers to understand the best practices for how to effectively apply test doubles.

我們已經瞭解到，測試替代對工程速度至關重要，因為它們可以幫助全面測試程式碼並確保測試快速執行。另一方面，誤用它們可能是生產力的主要消耗，因為它們可能導致測試不清楚、不可靠、效率較低。這就是為什麼工程師瞭解如何有效應用測試替代的最佳實踐非常重要。

There is often no exact answer regarding whether to use a real implementation or a test double, or which test double technique to use. An engineer might need to make some trade-offs when deciding the proper approach for their use case.

關於是使用真實實現還是測試替代，或者使用哪種測試替代技術，通常沒有確切的答案。工程師在為他們的用例決定合適的方法時可能需要做出一些權衡。

Although test doubles are great for working around dependencies that are difficult to use in tests, if you want to maximize confidence in your code, at some point you still want to exercise these dependencies in tests. The next chapter will cover larger-scope testing, for which these dependencies are used regardless of their suitability for unit tests; for example, even if they are slow or nondeterministic.

儘管測試替代對於處理測試中難以使用的依賴項非常有用，但如果你想最大限度地提高程式碼的可信度，在某些時候你仍然希望在測試中使用這些依賴項。下一章將介紹更大範圍的測試，對於這些測試，不管它們是否適合單元測試，都將使用這些依賴關係；例如，即使它們很慢或不確定。

## TL;DRs  內容提要
- A real implementation should be preferred over a test double.
- A fake is often the ideal solution if a real implementation can’t be used in a test.
- Overuse of stubbing leads to tests that are unclear and brittle.
- Interaction testing should be avoided when possible: it leads to tests that are brittle because it exposes implementation details of the system under test.

- 真實實現應優先於測試替代。
- 如果在測試中不能使用真實實現，那麼偽造實現通常是理想的解決方案。
- 過度使用打樁會導致測試不明確和變脆。
- 在可能的情況下，應避免互動測試：因為互動測試會暴露被測系統的實現細節，所以會導致測試不連貫。

