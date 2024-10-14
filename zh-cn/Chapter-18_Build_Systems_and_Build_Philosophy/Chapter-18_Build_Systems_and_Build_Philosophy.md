
**CHAPTER 18**

# Build Systems and Build Philosophy

# 第十八章 建立系統，建立理念

**Written by  Erik Kuefler**

**Edited by Lisa Carey**

If you ask Google engineers what they like most about working at Google (besides the free food and cool products), you might hear something surprising: engineers love the build system.[^1] Google has spent a tremendous amount of engineering effort over its lifetime in creating its own build system from the ground up, with the goal of ensuring that our engineers are able to quickly and reliably build code. The effort has been so successful that Blaze, the main component of the build system, has been reimplemented several different times by ex-Googlers who have left the company.[^2] In 2015, Google finally open sourced an implementation of Blaze named Bazel.

如果你問谷歌的工程師，他們最喜歡在谷歌工作的原因（除了免費的食物和黑科技產品），你還會聽到一些令人驚訝的事情：工程師們非常喜歡建立系統。谷歌一直在花費了巨大的努力，從零開始建立自己的建立系統，目的是確保工程師們能夠快速、可靠地建立程式碼。這一努力是成功的，建立系統的主要元件Blaze，已經被已離開公司的前谷歌員工重新實現了好幾次。2015年，谷歌終於公開了Blaze的一個實現，名為Bazel。

> [^1]: In an internal survey, 83% of Googlers reported being satisfied with the build system, making it the fourth most satisfying tool of the 19 surveyed. The average tool had a satisfaction rating of 69%.
>
> 1  在一項內部調查中，83%的谷歌使用者表示對建立系統感到滿意，這使其成為19個被調查工具中第四個最令人滿意的工具。平均工具的滿意度為69%。
>
> [^2]: See `https://buck.build/` and `https://www.pantsbuild.org/index.html`.
>
> 2 查閱 `https://buck.build/` and `https://www.pantsbuild.org/index.html`

## Purpose of a Build System 建立系統的目的

Fundamentally, all build systems have a straightforward purpose: they transform the source code written by engineers into executable binaries that can be read by machines. A good build system will generally try to optimize for two important properties:

*Fast*
    A developer should be able to type a single command to run the build and get back the resulting binary, often in as little as a few seconds.

*Correct*
    Every time any developer runs a build on any machine, they should get the same result (assuming that the source files and other inputs are the same).

從根上說，所有的建立系統都有一個簡單的目的：它們將工程師編寫的原始碼轉化為機器可以讀取的可執行二進位檔案。一個好的建立系統通常會試圖最佳化兩個重要的屬性：

*快*
    開發人員應該能夠輸入簡單的指令來執行建立並回傳生成的二進位檔案，而且只需幾秒鐘
*正確*
    任何開發人員在任何機器上執行建立，他們都應該得到相同的結果（假設源檔案和其他輸入是相同的）。

Many older build systems attempt to make trade-offs between speed and correctness by taking shortcuts that can lead to inconsistent builds. Bazel」s main objective is to avoid having to choose between speed and correctness, providing a build system structured to ensure that it」s always possible to build code efficiently and consistently.

許多較老的建立系統嘗試在速度和正確性之間做出權衡，採取了一些可能導致不一致的建立的捷徑。Bazel的主要目標是避免在速度和正確性之間做出選擇，提供一個結構化的建立系統，以確保總是可以高效和一致地建立程式碼。

Build systems aren」t just for humans; they also allow machines to create builds automatically, whether for testing or for releases to production. In fact, the large majority of builds at Google are triggered automatically rather than directly by engineers. Nearly all of our development tools tie into the build system in some way, giving huge amounts of value to everyone working on our codebase. Here」s a small sample of workflows that take advantage of our automated build system:

- Code is automatically built, tested, and pushed to production without any human intervention. Different teams do this at different rates: some teams push weekly, others daily, and others as fast as the system can create and validate new builds. (see Chapter 24).
- Developer changes are automatically tested when they」re sent for code review (see Chapter 19) so that both the author and reviewer can immediately see any build or test issues caused by the change.
- Changes are tested again immediately before merging them into the trunk, making it much more difficult to submit breaking changes.
- Authors of low-level libraries are able to test their changes across the entire codebase, ensuring that their changes are safe across millions of tests and binaries.
- Engineers are able to create large-scale changes (LSCs) that touch tens of thousands of source files at a time (e.g., renaming a common symbol) while still being able to safely submit and test those changes. We discuss LSCs in greater detail in Chapter 22.

建立系統不僅僅是為人類服務的；它們也允許機器自動建立建立，無論是用於測試還是用於發布到生產環境。事實上，谷歌的大部分建立都是自動觸發的，而不是由工程師點選觸發的。我們幾乎所有的開發工具都以某種方式與建立系統相結合，為每個在我們的程式碼庫上工作的人提供了巨大的價值。以下是利用我們的自動建立系統的一小部分工作流範例：

- 程式碼自動建立、測試並推送到生產環境，無需任何人工干預。不同的團隊以不同的頻率做這件事：有些團隊每週推送一次，有些團隊每天推送一次，有些團隊則以系統能夠建立和驗證新建立的速度推送。(見第24章）。
- 開發人員的更改在傳送給程式碼審查時自動進行測試（參見第19章），以便作者和審查人員都可以立即看到更改引起的任何建立或測試問題。。
- 在將修改合併到主幹中之前，會立即對其進行測試，這使得送出破壞性修改變得更加困難。
- 基礎庫的作者能夠在整個程式碼庫中測試他們的修改，確保他們的修改在數百萬的測試和二進位檔案中是安全的。
- 工程師們能夠建立大規模的變更（LSCs），同時觸及數以萬計的源檔案（例如，重新命名公共符號），同時仍然能夠安全地送出和測試這些修改。我們將在第22章中更詳細地討論LSCs。

All of this is possible only because of Google」s investment in its build system. Although Google might be unique in its scale, any organization of any size can realize similar benefits by making proper use of a modern build system. This chapter describes what Google considers to be a 『modern build system』 and how to use such systems.

所有這些都是由於谷歌對其建立系統的投入才得以實現。儘管谷歌的規模是獨一無二的，但任何規模的組織都可以透過正確使用現代建立系統實現類似的好處。本章介紹了Google認為的 "現代建立系統"以及如何使用這些系統。

## What Happens Without a Build System? 沒有建立系統會怎樣？

Build systems allow your development to scale. As we」ll illustrate in the next section, we run into problems of scaling without a proper build environment.

建立系統使你的開發可擴充套件。正如我們將在下一節說明的那樣，我們在沒有適當的建立環境的情況下會遇到擴充套件問題。

### But All I Need Is a Compiler! 但我所需要的只是一個編譯器!

The need for a build system might not be immediately obvious. After all, most of us probably didn」t use a build system when we were first learning to code—we probably started by invoking tools like gcc or javac directly from the command line, or the equivalent in an integrated development environment (IDE). As long as all of our source code is in the same directory, a command like this works fine:

```shell
javac *.java
```

對建立系統的需求可能不是很明顯。畢竟，我們中的大多數人在最初學習編碼時可能並沒有使用建立系統--我們可能一開始就直接從命令行中呼叫gcc或javac等工具，或者在整合開發環境（IDE）中呼叫相應的工具。只要我們所有的原始碼都在同一個目錄下，這樣的指令就能正常工作：

```shell
javac *.java
```

This instructs the Java compiler to take every Java source file in the current directory and turn it into a binary class file. In the simplest case, this is all that we need.

這指示Java編譯器把當前目錄下的每一個Java源檔案都變成一個二進位類別檔案。在最簡單的情況下，這就是我們所需要的。

However, things become more complicated quickly as soon as our code expands. javac is smart enough to look in subdirectories of our current directory to find code that we import. But it has no way of finding code stored in other parts of the filesystem (perhaps a library shared by several of our projects). It also obviously only knows how to build Java code. Large systems often involve different pieces written in a variety of programming languages with webs of dependencies among those pieces, meaning no compiler for a single language can possibly build the entire system.

然而，隨著程式碼的擴充套件，事情很快就會變得更加複雜。javac非常聰明，可以在我們當前目錄的子目錄中尋找我們匯入的程式碼。但它沒有辦法找到儲存在檔案系統其他地方的程式碼（也許是我們幾個專案共享的函式庫）。顯然，它只知道如何建立Java程式碼。大型系統通常涉及到用各種程式語言編寫的不同部分，這些部分之間存在著依賴關係，這意味著沒有一個單一語言的編譯器可以建立整個系統。

As soon as we end up having to deal with code from multiple languages or multiple compilation units, building code is no longer a one-step process. We now need to think about what our code depends on and build those pieces in the proper order, possibly using a different set of tools for each piece. If we change any of the dependencies, we need to repeat this process to avoid depending on stale binaries. For a codebase of even moderate size, this process quickly becomes tedious and error-prone.

一旦我們不得不處理來自多種語言或多個編譯單元的程式碼，建立程式碼就不再是一步到位的過程。我們現在需要考慮我們的程式碼依賴於什麼，並以適當的順序建立這些部分，可能為每個部分使用一套不同的工具。如果我們改變了任何依賴關係，我們需要重複這個過程，以避免依賴過時的二進位檔案。對於一個中等規模的程式碼庫來說，這個過程很快就會變得乏味，並且容易出錯。

The compiler also doesn」t know anything about how to handle external dependencies, such as third-party JAR files in Java. Often the best we can do without a build system is to download the dependency from the internet, stick it in a lib folder on the hard drive, and configure the compiler to read libraries from that directory. Over time, it」s easy to forget what libraries we put in there, where they came from, and whether they」re still in use. And good luck keeping them up to date as the library maintainers release new versions.

編譯器也不知道如何處理外部依賴關係，比如Java中的第三方JAR檔案。通常，在沒有建立系統的情況下，我們能做的最好的事情就是從網上下載依賴關係，把它放在硬碟上的lib資料夾裡，並設定編譯器從該目錄中讀取函式庫。隨著時間的推移，我們很容易忘記我們把哪些函式庫放在那裡，它們來自哪裡，以及它們是否仍在使用。而且，當函式庫的維護者發布新的版本時，要想讓它們保持最新的狀態，那就得靠運氣了。

### Shell Scripts to the Rescue? 來自shell指令碼的拯救？

Suppose that your hobby project starts out simple enough that you can build it using just a compiler, but you begin running into some of the problems described previously. Maybe you still don」t think you need a real build system and can automate away the tedious parts using some simple shell scripts that take care of building things in the correct order. This helps out for a while, but pretty soon you start running into even more problems:

- It becomes tedious. As your system grows more complex, you begin spending almost as much time working on your build scripts as on real code. Debugging shell scripts is painful, with more and more hacks being layered on top of one another.
- It」s slow. To make sure you weren」t accidentally relying on stale libraries, you have your build script build every dependency in order every time you run it. You think about adding some logic to detect which parts need to be rebuilt, but that sounds awfully complex and error prone for a script. Or you think about specifying which parts need to be rebuilt each time, but then you」re back to square one.
- Good news: it」s time for a release! Better go figure out all the arguments you need to pass to the jar command to make your final build. And remember how to upload it and push it out to the central repository. And build and push the documentation updates, and send out a notification to users. Hmm, maybe this calls for another script...
- Disaster! Your hard drive crashes, and now you need to recreate your entire system. You were smart enough to keep all of your source files in version control, but what about those libraries you downloaded? Can you find them all again and make sure they were the same version as when you first downloaded them? Your scripts probably depended on particular tools being installed in particular places — can you restore that same environment so that the scripts work again? What about all those environment variables you set a long time ago to get the compiler working just right and then forgot about?
- Despite the problems, your project is successful enough that you」re able to begin hiring more engineers. Now you realize that it doesn」t take a disaster for the previous problems to arise—you need to go through the same painful bootstrapping process every time a new developer joins your team. And despite your best efforts, there are still small differences in each person」s system. Frequently, what works on one person」s machine doesn」t work on another」s, and each time it takes a few hours of debugging tool paths or library versions to figure out where the difference is.
- You decide that you need to automate your build system. In theory, this is as simple as getting a new computer and setting it up to run your build script every night using cron. You still need to go through the painful setup process, but now you don」t have the benefit of a human brain being able to detect and resolve minor problems. Now, every morning when you get in, you see that last night」s build failed because yesterday a developer made a change that worked on their system but didn」t work on the automated build system. Each time it」s a simple fix, but it happens so often that you end up spending a lot of time each day discovering and applying these simple fixes.
- Builds become slower and slower as the project grows. One day, while waiting for a build to complete, you gaze mournfully at the idle desktop of your coworker, who is on vacation, and wish there were a way to take advantage of all that wasted computational power.

假設你的業餘專案開始時非常簡單，你可以只用一個編譯器來建立它，但你開始遇到前面描述的一些問題。也許你仍然認為你不需要一個真正的建立系統，可以使用一些簡單的shell指令碼來自動處理那些繁瑣的部分，這些指令碼負責按照正確的順序建立東西。這會有一段時間的幫助，但很快你就會遇到更多的問題：

- 它變得乏味了。隨著你的系統越來越複雜，你開始花在建立指令碼上的時間幾乎和真正的寫程式碼一樣多。除錯shell指令碼是很痛苦的，越來越多的"黑"操作操作被疊加在一起。
- 速度很慢。為了確保你沒有意外地依賴過時的函式庫，你讓你的建立指令碼在每次執行時按順序建立每個依賴。你可以考慮新增一些邏輯來檢測哪些部分需要重建，但這對於一個指令碼來說聽起來非常複雜而且容易出錯。或者你可以考慮每次指定哪些部分需要重建，但是你又回到了原點。
- 好訊息是：現在是發布的時候了! 最好弄清楚所有需要傳遞給jar指令以進行最終建立的引數。並記住如何上傳並推送到中央倉庫。建立並推送文件更新，並向用戶傳送通知。嗯，也許這需要另一個指令碼......
- 災難! 硬碟崩潰了，現在需要重新建立整個系統。你很聰明，把所有的源檔案都儲存在版本控制中，但是你下載的那些函式庫呢？你能重新找到它們，並確保它們和你第一次下載它們時的版本相同嗎？你的指令碼可能依賴於特定的工具被安裝在特定的地方--你能恢復同樣的環境，使指令碼再次工作嗎？那些你很久以前為了讓編譯器工作得恰到好處而設定的環境變數，後來又忘記了，怎麼辦？
- 儘管有這些問題，你的專案還是足於成功，以至於你能夠開始僱用更多的工程師。現在你意識到，不需要一場災難就會出現以前的問題--每次有新的開發人員加入你的團隊，你都需要經歷同樣痛苦的啟動過程。而且，儘管你做了最大的努力，每個人的系統還是有小的差異。通常，在一個人的機器上起作用的東西在另一個人的機器上不起作用，每次除錯工具路徑或庫版本都需要幾個小時才能找出差異所在。
- 你決定需要自動化建立系統。從理論上講，這就像買一台新的電腦並設定它每天晚上使用cron執行你的建立指令碼一樣簡單。你仍然需要經歷痛苦的設定過程，但現在你沒有了需要調式檢測和解決小問題的好處。現在，每天早上當你進去的時候，你會看到昨晚的建立失敗了，因為昨天一個開發者做了一個改變，這個改變在他們的系統上有效，但在自動建立系統上卻不起作用。每次都是一個簡單的修復，但它經常發生，以至於你每天都要花費大量時間來發現和應用這些簡單的修復。
- 隨著專案的發展，建立的速度越來越慢。有一天，在等待建立完成時，你哀怨地注視著正在度假的同事的閒置桌面，希望有一種方法可以充分利用以前的計算能力。

You」ve run into a classic problem of scale. For a single developer working on at most a couple hundred lines of code for at most a week or two (which might have been the entire experience thus far of a junior developer who just graduated university), a compiler is all you need. Scripts can maybe take you a little bit farther. But as soon as you need to coordinate across multiple developers and their machines, even a perfect build script isn」t enough because it becomes very difficult to account for the minor differences in those machines. At this point, this simple approach breaks down and it」s time to invest in a real build system.

你遇到了一個典型的規模問題。對於一個開發人員來說，一個編譯器就是你所需要的一切，他最多工作幾百行程式碼，最多工作一兩週（這可能是一個剛從大學畢業的初級開發人員迄今為止的全部經驗）。指令碼可能會讓你走得更遠一些。但是一旦你需要在多個開發人員和他們的機器之間進行協作，即使是一個完美的建立指令碼也是不夠的，因為很難解釋這些機器中的細微差異。在這一點上，這個簡單的方法崩潰了，是時候開發一個真正的建立系統了。

## Modern Build Systems 現代化的建立系統

Fortunately, all of the problems we started running into have already been solved many times over by existing general-purpose build systems. Fundamentally, they aren」t that different from the aforementioned script-based DIY approach we were working on: they run the same compilers under the hood, and you need to understand those underlying tools to be able to know what the build system is really doing. But these existing systems have gone through many years of development, making them far more robust and flexible than the scripts you might try hacking together yourself.

幸運的是，我們開始遇到的所有問題已經被現有的通用建立系統多次解決。從根本上說，它們與前面提到的基於指令碼的DIY方法沒有什麼不同：它們在後台執行相同的編譯器，你需要瞭解這些底層工具，才能瞭解建立系統真正在做什麼。但是這些現有的系統已經經歷了多年的開發，使得它們比你自己嘗試破解的指令碼更加健壯和靈活。

### It」s All About Dependencies 一切都是關於依賴關係

In looking through the previously described problems, one theme repeats over and over: managing your own code is fairly straightforward, but managing its dependencies is much more difficult (and Chapter 21 )is devoted to covering this problem in detail). There are all sorts of dependencies: sometimes there」s a dependency on a task (e.g., 『push the documentation before I mark a release as complete』), and sometimes there」s a dependency on an artifact (e.g., 『I need to have the latest version of the computer vision library to build my code』). Sometimes, you have internal dependencies on another part of your codebase, and sometimes you have external dependencies on code or data owned by another team (either in your organization or a third party). But in any case, the idea of 『I need that before I can have this』 is something that recurs repeatedly in the design of build systems, and managing dependencies is perhaps the most fundamental job of a build system.

在回顧之前描述的問題時，有一個主題反覆出現：管理你自己的程式碼是相當簡單的，但管理它的依賴關係要困難得多（[第21章]專門詳細介紹了這個問題）。有各種各樣的依賴關係：有時依賴於任務（例如，『在我將發布標記為完成之前推送文件』），有時依賴於構件（例如，『我需要最新版本的電腦視覺函式庫來建立程式碼』）。有時，你對你的程式碼庫的另一部分有內部依賴性，有時你對另一個團隊（在你的組織中或第三方）擁有的程式碼或資料有外部依賴性。但無論如何，"在我擁有這個之前，我需要那個"的想法在建立系統的設計中反覆出現，而管理依賴性也許是建立系統最基本的工作。

### Task-Based Build Systems 基於任務的建立系統

The shell scripts we started developing in the previous section were an example of a primitive task-based build system. In a task-based build system, the fundamental unit of work is the task. Each task is a script of some sort that can execute any sort of logic, and tasks specify other tasks as dependencies that must run before them. Most major build systems in use today, such as Ant, Maven, Gradle, Grunt, and Rake, are task based.

我們在上一節開始開發的shell指令碼是一個原始的基於任務的建立系統的範例。在基於任務的建立系統中，工作的基本單位是任務。每個任務都是某種型別的指令碼，可以執行任何型別的邏輯，任務將其他任務指定為必須在它們之前執行的依賴項。目前使用的大多數主要建立系統，如Ant、Maven、Gradle、Grunt和Rake，都是基於任務的。

Instead of shell scripts, most modern build systems require engineers to create buildfiles that describe how to perform the build. Take this example from the Ant manual:

大多數現代建立系統要求工程師建立描述如何執行建立的建立檔案，而不是shell指令碼。以Ant手冊中的這個例子為例：

``` XML
<project name="MyProject" default="dist" basedir=".">
<description>
simple example build file
</description>
<!-- set global properties for this build -->
<property name="src" location="src"/>
<property name="build" location="build"/>
<property name="dist" location="dist"/>

<target name="init">
<!-- Create the time stamp -->
<tstamp/>
<!-- Create the build directory structure used by compile -->
<mkdir dir="${build}"/>
</target>

<target name="compile" depends="init" description="compile the source">
<!-- Compile the Java code from ${src} into ${build} -->
<javac srcdir="${src}" destdir="${build}"/>
</target>

<target name="dist" depends="compile" description="generate the distribution">
<!-- Create the distribution directory -->
<mkdir dir="${dist}/lib"/>

<!-- Put everything in ${build} into the MyProject-${DSTAMP}.jar file -->
<jar jarfile="${dist}/lib/MyProject-${DSTAMP}.jar" basedir="${build}"/>
</target>

<target name="clean" description="clean up">
<!-- Delete the ${build} and ${dist} directory trees -->
<delete dir="${build}"/>
<delete dir="${dist}"/>
</target>
</project>
```

The buildfile is written in XML and defines some simple metadata about the build along with a list of tasks (the `<target>` tags in the XML[^3]). Each task executes a list of possible commands defined by Ant, which here include creating and deleting directories, running javac, and creating a JAR file. This set of commands can be extended by user-provided plug-ins to cover any sort of logic. Each task can also define the tasks it depends on via the depends attribute. These dependencies form an acyclic graph (see Figure 18-1).

建立檔案是用XML編寫的，定義了一些關於建立的簡單元資料以及任務清單（XML中的`<target>`標籤）。每個任務都執行Ant定義的一系列可能的指令，其中包括建立和刪除目錄、執行javac和建立JAR檔案。這組指令可以由使用者提供的外掛擴充套件，以涵蓋任何型別的邏輯。每個任務還可以透過依賴屬性定義它所依賴的任務。這些依賴關係形成一個無環圖（見圖18-1）。

Figure 18-1. An acyclic graph showing dependencies 顯示依賴關係的無環圖

![Figure 18-1](./images/Figure%2018-1.jpg)

Users perform builds by providing tasks to Ant」s command-line tool. For example, when a user types ant dist, Ant takes the following steps:

1. Loads a file named *build.xml* in the current directory and parses it to create the graph structure shown in Figure 18-1.
2. Looks for the task named dist that was provided on the command line and discovers that it has a dependency on the task named compile.
3. Looks for the task named compile and discovers that it has a dependency on the task named init.
4. Looks for the task named init and discovers that it has no dependencies.
5. Executes the commands defined in the init task.
6. Executes the commands defined in the compile task given that all of that task」s dependencies have been run.
7. Executes the commands defined in the dist task given that all of that task」s dependencies have been run.

使用者透過向Ant的指令行工具提供任務來執行建立。例如，當用戶輸入ant dist時，Ant會採取以下步驟:

1. 在當前目錄下載入一個名為*build.xml*的檔案，並對其進行解析以建立圖18-1所示的圖結構。
2. 尋找指令行上提供的名為dist的任務，並發現它與名為compile的任務有依賴關係。
3. 尋找名為compile的任務，發現它與名為init的任務有依賴關係。
4. 查詢名為init的任務並確認它沒有依賴項。
5. 執行init任務中定義的指令。
6. 執行編譯任務中定義的指令，前提是該任務的所有依賴項都已執行。
7. 執行dist任務中定義的指令，前提是該任務的所有依賴項都已執行。

In the end, the code executed by Ant when running the dist task is equivalent to the following shell script:

最後，Ant在執行dist任務時執行的程式碼相當於以下shell指令碼：

```shell
./createTimestamp.sh 
mkdir build/
javac src/* -d build/
mkdir -p dist/lib/
jar cf dist/lib/MyProject-$(date --iso-8601).jar build/*
```

When the syntax is stripped away, the buildfile and the build script actually aren」t too different. But we」ve already gained a lot by doing this. We can create new buildfiles in other directories and link them together. We can easily add new tasks that depend on existing tasks in arbitrary and complex ways. We need only pass the name of a single task to the ant command-line tool, and it will take care of determining everything that needs to be run.

去掉語法後，建立檔案和建立指令碼實際上沒有太大區別。但我們這樣做已經有了很大的收穫。我們可以在其他目錄中建立新的建立檔案並將它們連結在一起。我們可以以任意和複雜的方式輕鬆新增依賴於現有任務的新任務。我們只需要將單個任務的名稱傳遞給ant指令行工具，它將負責確定需要執行的所有內容。

Ant is a very old piece of software, originally released in 2000—not what many people would consider a 『modern』 build system today! Other tools like Maven and Gradle have improved on Ant in the intervening years and essentially replaced it by adding features like automatic management of external dependencies and a cleaner syntax without any XML. But the nature of these newer systems remains the same: they allow engineers to write build scripts in a principled and modular way as tasks and provide tools for executing those tasks and managing dependencies among them.

Ant是一個非常古老的軟體，最初發布於2000年--而不是很多人今天會考慮的『現代』建立系統！其他工具，如Maven和Gradle，在這幾年中對Ant進行了改進，基本上取代了它，新增諸如自動管理外部依賴項和不使用任何XML的更乾淨語法等功能。但這些新系統的本質仍然是一樣的：它們允許工程師以有原則的模組化方式編寫建立指令碼作為任務，並提供工具來執行這些任務和管理它們之間的依賴關係。

> [^3]:  Ant uses the word 『target』 to represent what we call a 『task』 in this chapter, and it uses the word 『task』 to refer to what we call 『commands.』
>
> 3 ant用 "目標 "這個詞來表示我們在本章中所說的 "任務"，它用 "任務 "這個詞來指代我們所說的 "指令"/。

#### The dark side of task-based build systems 基於任務的建立系統的缺陷

Because these tools essentially let engineers define any script as a task, they are extremely powerful, allowing you to do pretty much anything you can imagine with them. But that power comes with drawbacks, and task-based build systems can become difficult to work with as their build scripts grow more complex. The problem with such systems is that they actually end up giving *too much power to engineers and not enough power to the system*. Because the system has no idea what the scripts are doing, performance suffers, as it must be very conservative in how it schedules and executes build steps. And there」s no way for the system to confirm that each script is doing what it should, so scripts tend to grow in complexity and end up being another thing that needs debugging.

因為這些工具本質上允許工程師將任何指令碼定義為一項任務，所以它們非常強大，允許你用它們做幾乎任何你能想象到的事情。但是，這種能力也有缺點，基於任務的建立系統會隨著建立指令碼的日益複雜而變得難以使用。這類系統的問題是，它們實際上最終把*過多的權力給工程師，而沒有把足夠的權力給系統*。因為系統不知道指令碼在做什麼，效能受到影響，因為它在排程和執行建立步驟時必須非常保守。而且，系統無法確認每個指令碼都在做它應該做的事情，因此指令碼往往會變得越來越複雜，最終成為另一件需要除錯的事情。

**Difficulty of parallelizing build steps.** Modern development workstations are typically quite powerful, with multiple cores that should theoretically be capable of executing several build steps in parallel. But task-based systems are often unable to parallelize task execution even when it seems like they should be able to. Suppose that task A depends on tasks B and C. Because tasks B and C have no dependency on each other, is it safe to run them at the same time so that the system can more quickly get to task A? Maybe, if they don」t touch any of the same resources. But maybe not—perhaps both use the same file to track their statuses and running them at the same time will cause a conflict. There」s no way in general for the system to know, so either it has to risk these conflicts (leading to rare but very difficult-to-debug build problems), or it has to restrict the entire build to running on a single thread in a single process. This can be a huge waste of a powerful developer machine, and it completely rules out the possibility of distributing the build across multiple machines.

**並行化建立步驟的難點。** 現代開發工作站通常非常強大，有多個CPU核心，理論上應該能夠並行執行幾個建立步驟。但是，基於任務的系統往往無法將任務執行並行化，即使是在看起來應該能夠做到的時候。假設任務A依賴於任務B和C。因為任務B和C彼此不依賴，所以同時執行它們是否安全，以便系統可以更快地到達任務A？也許吧，如果它們不接觸任何相同的資源。但也許不是--也許它們都使用同一個檔案來跟蹤它們的狀態，同時執行它們會導致衝突。一般來說，系統無法知道，所以要麼它不得不冒著這些衝突的風險（導致罕見但非常難以除錯的建立問題），要麼它必須限制整個建立在單個程序的單個執行緒上執行。這可能是對強大的開發者機器的巨大浪費，而且它完全排除了在多台機器上分佈建立的可能性。

**Difficulty performing incremental builds**. A good build system will allow engineers to perform reliable incremental builds such that a small change doesn」t require the entire codebase to be rebuilt from scratch. This is especially important if the build system is slow and unable to parallelize build steps for the aforementioned reasons. But unfortunately, task-based build systems struggle here, too. Because tasks can do anything, there」s no way in general to check whether they」ve already been done. Many tasks simply take a set of source files and run a compiler to create a set of binaries; thus, they don」t need to be rerun if the underlying source files haven」t changed. But without additional information, the system can」t say this for sure—maybe the task downloads a file that could have changed, or maybe it writes a timestamp that could be different on each run. To guarantee correctness, the system typically must rerun every task during each build.

**難以執行增量建立。** 一個好的建立系統將允許工程師執行可靠的增量建立，這樣，一個小的變更就不需要從頭開始重建整個程式碼庫了。如果建立系統由於上述原因，速度很慢，無法並行化建立步驟，那麼這一點就尤為重要。但不幸的是，基於任務的建立系統在這裡也很困難。因為任務可以做任何事情，一般來說，沒有辦法檢查它們是否已經完成。許多任務只是接收一組源檔案並執行一個編譯器來建立一組二進位檔案；因此，如果底層源檔案沒有更改，則不需要重新執行。但是，如果沒有額外的訊息，系統就不能確定這一點--可能是任務下載了一個可能已更改的檔案，或者它在每次執行時寫入了一個可能不同的時間戳。為了保證正確性，系統通常必須在每次建立期間重新執行每個任務。

Some build systems try to enable incremental builds by letting engineers specify the conditions under which a task needs to be rerun. Sometimes this is feasible, but often it」s a much trickier problem than it appears. For example, in languages like C++ that allow files to be included directly by other files, it」s impossible to determine the entire set of files that must be watched for changes without parsing the input sources. Engineers will often end up taking shortcuts, and these shortcuts can lead to rare and frustrating problems where a task result is reused even when it shouldn」t be. When this happens frequently, engineers get into the habit of running clean before every build to get a fresh state, completely defeating the purpose of having an incremental build in the first place. Figuring out when a task needs to be rerun is surprisingly subtle, and is a job better handled by machines than humans.

一些建立系統試圖透過讓工程師指定需要重新執行任務的條件來啟用增量建立。有時這是可行的，但通常這是一個比看起來更棘手的問題。例如，在像C++這樣允許檔案直接被其他檔案包含的語言中，如果不解析輸入源，就不可能確定必須關注的整個檔案集的變化。工程師們最終往往會走捷徑，而這些捷徑會導致罕見的、令人沮喪的問題，即一個任務結果被重複使用，即使它不應該被使用。當這種情況經常發生時，工程師們就會養成習慣，在每次建立前執行clean，以獲得一個全新的狀態，這就完全違背了一開始就有增量建立的目的。弄清楚什麼時候需要重新執行一個任務是非常微妙的，而且是一個最好由機器而不是人處理的工作。

**Difficulty maintaining and debugging scripts**. Finally, the build scripts imposed by task- based build systems are often just difficult to work with. Though they often receive less scrutiny, build scripts are code just like the system being built, and are easy places for bugs to hide. Here are some examples of bugs that are very common when working with a task-based build system:

- Task A depends on task B to produce a particular file as output. The owner of task B doesn」t realize that other tasks rely on it, so they change it to produce output in a different location. This can」t be detected until someone tries to run task A and finds that it fails.
- Task A depends on task B, which depends on task C, which is producing a particular file as output that」s needed by task A. The owner of task B decides that it doesn」t need to depend on task C any more, which causes task A to fail even though task B doesn」t care about task C at all!
- The developer of a new task accidentally makes an assumption about the machine running the task, such as the location of a tool or the value of particular environment variables. The task works on their machine, but fails whenever another developer tries it.
- A task contains a nondeterministic component, such as downloading a file from the internet or adding a timestamp to a build. Now, people will get potentially different results each time they run the build, meaning that engineers won」t always be able to reproduce and fix one another」s failures or failures that occur on an automated build system.
- Tasks with multiple dependencies can create race conditions. If task A depends on both task B and task C, and task B and C both modify the same file, task A will get a different result depending on which one of tasks B and C finishes first.

**難以維護和除錯指令碼**。最後，基於任務的建立系統所強加的建立指令碼往往就是難以使用。儘管建立指令碼通常很少受到審查，但它們與正在建立的系統一樣，都是程式碼，很容易隱藏bug。以下是使用基於任務的建立系統時常見的一些錯誤範例：

- 任務A依賴於任務B來產生一個特定的檔案作為輸出。任務B的所有者沒有意識到其他任務依賴於它，所以他們改變了它，在不同的位置產生輸出。直到有人試圖執行任務A，發現它失敗了，這才被發現。
- 任務A依賴於任務B，而任務B依賴於任務C，而任務C正在產生一個任務A需要的特定檔案作為輸出。任務B的所有者決定它不需要再依賴於任務C，這導致任務A失敗，儘管任務B根本不關心任務C!
- 一個新任務的開發者不小心對執行該任務的機器做了一個設定，比如一個工具的位置或特定環境變數的值。該任務在他們的機器上可以執行，但只要其他開發者嘗試，就會失敗。
- 任務包含不確定元件，例如從internet下載檔案或向生成新增時間戳。現在，人們每次執行建立時都會得到可能不同的結果，這意味著工程師不可能總是能夠重現和修復彼此的故障或自動建立系統上發生的故障。
- 有多個依賴關係的任務會產生競賽條件。如果任務A同時依賴於任務B和任務C，而任務B和任務C同時修改同一個檔案，那麼任務A會得到不同的結果，這取決於任務B和任務C中哪一個先完成。

There」s no general-purpose way to solve these performance, correctness, or maintainability problems within the task-based framework laid out here. So long as engineers can write arbitrary code that runs during the build, the system can」t have enough information to always be able to run builds quickly and correctly. To solve the problem, we need to take some power out of the hands of engineers and put it back in the hands of the system and reconceptualize the role of the system not as running tasks, but as producing artifacts. This is the approach that Google takes with Blaze and Bazel, and it will be described in the next section.

在這裡列出的基於任務的框架中，沒有通用的方法來解決這些效能、正確性或可維護性問題。只要工程師能夠編寫在建立過程中執行的任意程式碼，系統就不可能擁有足夠的訊息來始終能夠快速、正確地執行建立。我們需要從工程師手中奪走一些權力，把它放回系統的手中，並重新認識到系統的作用不是作為執行任務，而是作為生產元件。這就是谷歌對Blaze和Bazel採取的方法，將在下一節進行描述。

### Artifact-Based Build Systems 基於構件的建立系統

To design a better build system, we need to take a step back. The problem with the earlier systems is that they gave too much power to individual engineers by letting them define their own tasks. Maybe instead of letting engineers define tasks, we can have a small number of tasks defined by the system that engineers can configure in a limited way. We could probably deduce the name of the most important task from the name of this chapter: a build system」s primary task should be to build code. Engineers would still need to tell the system what to build, but the how of doing the build would be left to the system.

為了設計一個更好的建立系統，我們需要後退一步。早期系統的問題在於，它們讓工程師定義自己的任務，從而給了他們太多的權力。也許，我們可以不讓工程師定義任務，而是由系統定義少量的任務，讓工程師以有限的方式進行設定。我們也許可以從本章的名稱中推斷出最重要的任務的名稱：建立系統的主要任務應該是建立程式碼。工程師們仍然需要告訴系統要建立什麼，但如何建立的問題將留給系統。

This is exactly the approach taken by Blaze and the other artifact-based build systems descended from it (which include Bazel, Pants, and Buck). Like with task-based build systems, we still have buildfiles, but the contents of those buildfiles are very different. Rather than being an imperative set of commands in a Turing-complete scripting language describing how to produce an output, buildfiles in Blaze are a declarative manifest describing a set of artifacts to build, their dependencies, and a limited set of options that affect how they」re built. When engineers run blaze on the command line, they specify a set of targets to build (the 『what』), and Blaze is responsible for configuring, running, and scheduling the compilation steps (the 『how』). Because the build system now has full control over what tools are being run when, it can make much stronger guarantees that allow it to be far more efficient while still guaranteeing correctness.

這正是Blaze和它衍生的其他基於構件的建立系統（包括Bazel、Pants和Buck）所採用的方法。與基於任務的建立系統一樣，我們仍然有建立檔案，但這些建立檔案的內容卻非常不同。在Blaze中，建立檔案不是圖靈完備的指令碼語言中描述如何產生輸出的指令集，而是宣告性的清單，描述一組要建立的構件、它們的依賴關係，以及影響它們如何建立的有限選項集。當工程師在指令行上執行blaze時，他們指定一組要建立的目標（"what"），而Blaze負責設定、執行和排程編譯步驟（"how"）。由於建立系統現在可以完全控制什麼工具在什麼時候執行，它可以做出更有力的保證，使其在保證正確性的同時，效率也大大提高。

#### A functional perspective 功能視角

It」s easy to make an analogy between artifact-based build systems and functional programming. Traditional imperative programming languages (e.g., Java, C, and Python) specify lists of statements to be executed one after another, in the same way that task- based build systems let programmers define a series of steps to execute. Functional programming languages (e.g., Haskell and ML), in contrast, are structured more like a series of mathematical equations. In functional languages, the programmer describes a computation to perform, but leaves the details of when and exactly how that computation is executed to the compiler. This maps to the idea of declaring a manifest in an artifact-based build system and letting the system figure out how to execute the build.

在基於構件的建立系統和函式式程式設計之間做個類比是很容易的。傳統的指令式程式語言（如Java、C和Python）指定了一個又一個要執行的語句清單，就像基於任務的建立系統讓工程師定義一系列的執行步驟一樣。相比之下，函式式程式語言（如Haskell和ML）的結構更像是一系列的數學方程。在函式式語言中，工程師描述了一個要執行的計算，但把何時以及如何執行該計算的細節留給了編譯器。這就相當於在基於構件的建立系統中宣告一個清單，並讓系統找出如何執行建立的思路。

Many problems cannot be easily expressed using functional programming, but the ones that do benefit greatly from it: the language is often able to trivially parallelize such programs and make strong guarantees about their correctness that would be impossible in an imperative language. The easiest problems to express using functional programming are the ones that simply involve transforming one piece of data into another using a series of rules or functions. And that」s exactly what a build system is: the whole system is effectively a mathematical function that takes source files (and tools like the compiler) as inputs and produces binaries as outputs. So, it」s not surprising that it works well to base a build system around the tenets of functional programming.

許多問題無法用函式式程式設計便捷表達，但那些確實從中受益匪淺的問題：函式式語言通常能夠簡單地並行這些程式，並對它們的正確性做出強有力的保證，而這在指令式語言中是不可能的。使用函式程式設計最容易表達的問題是使用一系列規則或函式將一段資料轉換為另一段數據的問題。而這正是建立系統的特點：整個系統實際上是一個數學函式，它將源檔案（和編譯器等工具）作為輸入，併產生二進位檔案作為輸出。因此，圍繞函式式程式設計的原則建立一個建立系統並不令人驚訝。

Getting concrete with Bazel. Bazel is the open source version of Google」s internal build tool, Blaze, and is a good example of an artifact-based build system. Here」s what a buildfile (normally named BUILD) looks like in Bazel:

用Bazel來實現具體化。Bazel是谷歌內部建立工具Blaze的開源版本，是基於構件的建立系統的一個好例子。下面是Bazel中建立檔案（通常名為BUILD）的內容：

```make
java_binary(
name = "MyBinary",
srcs = ["MyBinary.java"], deps = [
":mylib",
],
)

java_library(
name = "mylib",
srcs = ["MyLibrary.java", "MyHelper.java"],
visibility = ["//java/com/example/myproduct: subpackages "], deps = [
"//java/com/example/common", "//java/com/example/myproduct/otherlib", "@com_google_common_guava_guava//jar",
],
)
```

In Bazel, BUILD files define targets—the two types of targets here are java_binary and java_library. Every target corresponds to an artifact that can be created by the system: binary targets produce binaries that can be executed directly, and library targets produce libraries that can be used by binaries or other libraries. Every target has a name (which defines how it is referenced on the command line and by other targets, srcs (which define the source files that must be compiled to create the artifact for the target), and deps (which define other targets that must be built before this target and linked into it). Dependencies can either be within the same package (e.g., MyBinary」s dependency on ":mylib"), on a different package in the same source hierarchy (e.g., mylib」s dependency on "//java/com/example/common"), or on a third- party artifact outside of the source hierarchy (e.g., mylib」s dependency on "@com_google_common_guava_guava//jar"). Each source hierarchy is called a workspace and is identified by the presence of a special WORKSPACE file at the root.

在Bazel中，BUILD檔案定義了目標--這裡的兩類目標是java_binary和java_library。每個目標都對應於系統可以建立的構件：二進位目標產生可以直接執行的二進位檔案，而函式庫目標產生可以被二進位檔案或其他函式庫使用的函式庫。每個目標都有一個名字（它定義了它在指令行和其他目標中的引用方式）、srcs（它定義了必須被編譯以建立目標的元件的源檔案）和deps（它定義了必須在這個目標之前建立並連結到它的其他目標）。依賴關係可以是在同一個套件內（例如，MyBinary對":mylib "的依賴），也可以是在同一個源層次結構中的不同套件上（例如，mylib對"//java/com/example/common "的依賴），或者是在源層次結構之外的第三方構件上（例如，mylib對"@com_google_common_guava_guava//jar "的依賴）。每個源層次結構被稱為工作區，並透過在根部存在一個特殊的WORKSPACE檔案來識別。

Like with Ant, users perform builds using Bazel」s command-line tool. To build the MyBinary target, a user would run bazel build :MyBinary. Upon entering that command for the first time in a clean repository, Bazel would do the following:  

1. Parse every BUILD file in the workspace to create a graph of dependencies among artifacts.
2. Use the graph to determine the transitive dependencies of MyBinary; that is, every target that MyBinary depends on and every target that those targets depend on, recursively.  
3. Build (or download for external dependencies) each of those dependencies, in order. Bazel starts by building each target that has no other dependencies and keeps track of which dependencies still need to be built for each target. As soon as all of a target」s dependencies are built, Bazel starts building that target. This process continues until every one of MyBinary」s transitive dependencies have been built.
4. Build MyBinary to produce a final executable binary that links in all of the dependencies that were built in step 3.
    Fundamentally, it might not seem like what」s happening here is that much different than what happened when using a task-based build system. Indeed, the end result is the same binary, and the process for producing it involved analyzing a bunch of steps to find dependencies among them, and then running those steps in order. But there are critical differences. The first one appears in step 3: because Bazel knows that each target will only produce a Java library, it knows that all it has to do is run the Java compiler rather than an arbitrary user-defined script, so it knows that it」s safe to run these steps in parallel. This can produce an order of magnitude performance improvement over building targets one at a time on a multicore machine, and is only possible because the artifact-based approach leaves the build system in charge of its own execution strategy so that it can make stronger guarantees about parallelism.

和Ant一樣，使用者使用Bazel的指令行工具進行建立。為了建立MyBinary目標，使用者可以執行 bazel build :MyBinary。在一個乾淨的版本庫中第一次輸入該指令時，Bazel會做以下工作。

  1. 解析工作區中的每個BUILD檔案，以建立構件之間的依賴關係圖。
  2. 使用該圖來確定MyBinary的橫向依賴關係；也就是說，MyBinary所依賴的每個目標以及這些目標所依賴的每個目標都是遞迴的。
  3. 生成（或下載外部依賴項）每個依賴項按順序排列。Bazel首先建立沒有其他依賴項的每個目標，並跟蹤每個目標仍需要建立哪些依賴項。一旦建立了目標的所有依賴項，Bazel就會開始建立該目標。此過程一直持續到MyBinary的每個可傳遞依賴項已經建成。
  4. 建立MyBinary，產生一個最終的可執行二進位檔案，該檔案連結了在步驟3中建立的所有依賴項。

The benefits extend beyond parallelism, though. The next thing that this approach gives us becomes apparent when the developer types bazel build :MyBinary a second time without making any changes: Bazel will exit in less than a second with a message saying that the target is up to date. This is possible due to the functional programming paradigm we talked about earlier—Bazel knows that each target is the result only of running a Java compiler, and it knows that the output from the Java compiler depends only on its inputs, so as long as the inputs haven」t changed, the output can be reused. And this analysis works at every level; if MyBinary.java changes, Bazel knows to rebuild MyBinary but reuse mylib. If a source file for //java/com/ example/common changes, Bazel knows to rebuild that library, mylib, and MyBinary, but reuse //java/com/example/myproduct/otherlib. Because Bazel knows about the properties of the tools it runs at every step, it」s able to rebuild only the minimum set of artifacts each time while guaranteeing that it won」t produce stale builds.

從根本上說，這裡發生的事情似乎與使用基於任務的建立系統時發生的事情沒有太大的不同。事實上，最終結果是相同的二進位檔案，生成它的過程包括分析一系列步驟以找到它們之間的依賴關係，然後按順序執行這些步驟。但是有一些關鍵的區別。第一個出現在第3步：因為Bazel知道每個目標只會生成一個Java函式庫，所以它知道它所要做的就是執行Java編譯器，而不是任意的使用者定義指令碼，所以它知道執行它是安全的。這些步驟是並行的。與在多核機器上一次建立一個目標相比，這可以產生一個數量級的效能改進，並且這是唯一可能的，因為基於構件的方法讓建立系統負責自己的執行策略，以便它能夠對並行性做出更有力的保證。

Reframing the build process in terms of artifacts rather than tasks is subtle but powerful. By reducing the flexibility exposed to the programmer, the build system can know more about what is being done at every step of the build. It can use this knowledge to make the build far more efficient by parallelizing build processes and reusing their outputs. But this is really just the first step, and these building blocks of parallelism and reuse will form the basis for a distributed and highly scalable build system that will be discussed later.

從構件而不是任務的角度來重構建立過程是微妙而強大的。透過減少暴露在工程師面前的靈活性，建立系統可以知道更多關於在建立的每一步正在做什麼。它可以利用這些知識，透過並行化建立過程和重用其輸出，使建立的效率大大提升。但這實際上只是第一步，這些並行和重用的構件將構成分散式和高度可擴充套件的建立系統的基礎，這將在後面討論。

### Other nifty Bazel tricks 其他有趣的Bazel技巧

Artifact-based build systems fundamentally solve the problems with parallelism and reuse that are inherent in task-based build systems. But there are still a few problems that came up earlier that we haven」t addressed. Bazel has clever ways of solving each of these, and we should discuss them before moving on.

基於構件的建立系統從根本上解決了基於任務的建立系統所固有的並行性和重用問題。但仍有一些問題在前面出現過，我們還沒有解決。Bazel有解決這些問題的聰明方法，我們應該在繼續之前討論它們。

**Tools as dependencies**. One problem we ran into earlier was that builds depended on the tools installed on our machine, and reproducing builds across systems could be difficult due to different tool versions or locations. The problem becomes even more difficult when your project uses languages that require different tools based on which platform they」re being built on or compiled for (e.g., Windows versus Linux), and each of those platforms requires a slightly different set of tools to do the same job.

**工具作為依賴項**。我們之前遇到的一個問題是，建立取決於我們機器上安裝的工具，由於工具版本或位置不同，跨系統複製建立可能會很困難。當你的專案使用的語言需要根據它們在哪個平台上建立或編譯的不同工具時（例如，Windows與Linux），這個問題就變得更加困難，而每個平台都需要一套稍微不同的工具來完成同樣的工作。

Bazel solves the first part of this problem by treating tools as dependencies to each target. Every java_library in the workspace implicitly depends on a Java compiler, which defaults to a well-known compiler but can be configured globally at the workspace level. Whenever Blaze builds a java_library, it checks to make sure that the specified compiler is available at a known location and downloads it if not. Just like any other dependency, if the Java compiler changes, every artifact that was dependent upon it will need to be rebuilt. Every type of target defined in Bazel uses this same strategy of declaring the tools it needs to run, ensuring that Bazel is able to bootstrap them no matter what exists on the system where it runs.

Bazel解決了這個問題的第一部分，把工具當作對每個目標的依賴。工作區中的每一個java_library都隱含地依賴於一個Java編譯器，它預設為一個知名的編譯器，但可以在工作區層面進行全域設定。每當Blaze建立一個java_library時，它都會檢查以確保指定的編譯器在已知的位置上是可用的，如果不可用，就下載它。就像其他依賴關係一樣，如果Java編譯器改變了，每個依賴它的構件都需要重建。在Bazel中定義的每一種型別的目標都使用這種相同的策略來宣告它需要執行的工具，確保Bazel能夠啟動它們，無論它執行的系統上存在什麼。

Bazel solves the second part of the problem, platform independence, by using toolchains. Rather than having targets depend directly on their tools, they actually depend on types of toolchains. A toolchain contains a set of tools and other properties defining how a type of target is built on a particular platform. The workspace can define the particular toolchain to use for a toolchain type based on the host and target platform. For more details, see the Bazel manual.

Bazel透過使用工具鏈解決了問題的第二部分，即平台獨立性。與其讓目標直接依賴於它們的工具，不如說它們實際上依賴於工具鏈的型別。工具鏈包含一組工具和其他屬性，定義了如何在特定平台上建立目標型別。工作區可以定義基於主機和目標平台，為工具鏈型別使用特定的工具鏈。有關更多詳細訊息，請參閱Bazel手冊。

**Extending the build system**. Bazel comes with targets for several popular programming languages out of the box, but engineers will always want to do more—part of the benefit of task-based systems is their flexibility in supporting any kind of build process, and it would be better not to give that up in an artifact-based build system. Fortunately, Bazel allows its supported target types to be extended by adding custom rules.

**擴充套件建立系統**。Bazel為幾種流行的程式語言提供了開箱即用的能力，但工程師們總是想做得更多--基於任務的系統的部分好處是它們在支援任何型別的建立過程中的靈活性，在基於構件的建立系統中最好也可以支援這一點。幸運的是，Bazel允許其支援透過新增自定義規則擴充套件的目標類別型。

To define a rule in Bazel, the rule author declares the inputs that the rule requires (in the form of attributes passed in the BUILD file) and the fixed set of outputs that the rule produces. The author also defines the actions that will be generated by that rule. Each action declares its inputs and outputs, runs a particular executable or writes a particular string to a file, and can be connected to other actions via its inputs and outputs. This means that actions are the lowest-level composable unit in the build system —an action can do whatever it wants so long as it uses only its declared inputs and outputs, and Bazel will take care of scheduling actions and caching their results as appropriate.

要在Bazel中定義規則，規則作者要宣告該規則需要的輸入（以BUILD檔案中傳遞的屬性形式）和該規則產生的固定輸出集。作者還定義了將由該規則生成的操作。每個操作都宣告其輸入和輸出，執行特定的可執行檔案或將特定字串寫入檔案，並可以透過其輸入和輸出連線到其他操作。這意味著操作是建立系統中最底層的可組合單元--一個操作可以做任何它想做的事情，只要它只使用它所宣告的輸入和輸出，Bazel將負責排程動作並適當地快取其結果。

The system isn」t foolproof given that there」s no way to stop an action developer from doing something like introducing a nondeterministic process as part of their action. But this doesn」t happen very often in practice, and pushing the possibilities for abuse all the way down to the action level greatly decreases opportunities for errors. Rules supporting many common languages and tools are widely available online, and most projects will never need to define their own rules. Even for those that do, rule definitions only need to be defined in one central place in the repository, meaning most engineers will be able to use those rules without ever having to worry about their implementation.

這個系統並不是萬無一失的，因為沒有辦法阻止操作開發者做一些事情，比如在他們的操作中引入一個不確定的過程。但這種情況在實踐中並不經常發生，而且將濫用的可能性一直推到操作層面，大大減少了錯誤的機會。支援許多常用語言和工具的規則在網上廣泛提供，大多數專案都不需要定義自己的規則。即使是那些需要定義規則的專案，規則定義也只需要在儲存庫中的一個中心位置定義，這意味著大多數工程師將能夠使用這些規則，而不必擔心它們的實現。

**Isolating the environment**. Actions sound like they might run into the same problems as tasks in other systems—isn」t it still possible to write actions that both write to the same file and end up conflicting with one another? Actually, Bazel makes these conflicts impossible by using sandboxing. On supported systems, every action is isolated from every other action via a filesystem sandbox. Effectively, each action can see only a restricted view of the filesystem that includes the inputs it has declared and any outputs it has produced. This is enforced by systems such as LXC on Linux, the same technology behind Docker. This means that it」s impossible for actions to conflict with one another because they are unable to read any files they don」t declare, and any files that they write but don」t declare will be thrown away when the action finishes. Bazel also uses sandboxes to restrict actions from communicating via the network.

**隔離環境**。行動聽起來可能會遇到與其他系統中的任務相同的問題--難道沒有可能寫入同時寫入同一檔案並最終相互衝突的操作嗎？實際上，Bazel透過使用沙箱使這些衝突變得不可能。在支援的系統上，每個操作都透過檔案系統沙盒與其他動作隔離開來。實際上，每個操作只能看到檔案系統的一個有限檢視，包括它所宣告的輸入和它生成的任何輸出。這是由Linux上的LXC等系統強制執行的，Docker背後的技術也是如此。這意味著操作之間不可能發生衝突，因為它們無法讀取它們沒有宣告的任何檔案，並且他們編寫但未宣告的檔案將在操作完成時被丟棄。Bazel還使用沙盒來限制行動透過網路進行通訊。

**Making external dependencies deterministic**. There」s still one problem remaining: build systems often need to download dependencies (whether tools or libraries) from external sources rather than directly building them. This can be seen in the example via the @com_google_common_guava_guava//jar dependency, which downloads a JAR file from Maven.

**使外部依賴性具有確定性**。還有一個問題：建立系統經常需要從外部下載依賴項（無論是工具還是函式庫），而不是直接建立它們。這可以透過@com_google_common_guava_guava//jar依賴項在範例中看到，該依賴項從Maven下載jar檔案。

Depending on files outside of the current workspace is risky. Those files could change at any time, potentially requiring the build system to constantly check whether they」re fresh. If a remote file changes without a corresponding change in the workspace source code, it can also lead to unreproducible builds—a build might work one day and fail the next for no obvious reason due to an unnoticed dependency change. Finally, an external dependency can introduce a huge security risk when it is owned by a third party:[^4]  if an attacker is able to infiltrate that third-party server, they can replace the dependency file with something of their own design, potentially giving them full control over your build environment and its output.

依靠當前工作區以外的檔案是有風險的。這些檔案可能隨時更改，這可能需要生成系統不斷檢查它們是否是最新的。如果一個遠端檔案發生了變化，而工作區的原始碼卻沒有相應的變化，這也會導致建立的不可重複性--由於一個未被注意到的依賴性變化，建立可能在某一天成功，而在第二天卻沒有明顯的原因而失敗。最後，當外部依賴項屬於第三方時，可能會帶來巨大的安全風險：如果攻擊者能夠滲透到第三方伺服器，他們可以用自己設計的內容替換依賴項檔案，從而有可能讓他們完全控制伺服器建立環境及其輸出。

The fundamental problem is that we want the build system to be aware of these files without having to check them into source control. Updating a dependency should be a conscious choice, but that choice should be made once in a central place rather than managed by individual engineers or automatically by the system. This is because even with a 『Live at Head』 model, we still want builds to be deterministic, which implies that if you check out a commit from last week, you should see your dependencies as they were then rather than as they are now.

根本的問題是，我們希望建立系統知道這些檔案，而不必將它們放入原始碼管理。更新一個依賴關係應該是一個有意識的選擇，但這個選擇應該在一個中心位置做出，而不是由個別工程師管理或由系統自動管理。這是因為即使是 "Live at Head "模式，我們仍然希望建立是確定性的，這意味著如果你檢查出上周的送出，你應該看到你的依賴關係是當時的，而不是現在的。

Bazel and some other build systems address this problem by requiring a workspace- wide manifest file that lists a cryptographic hash for every external dependency in the workspace.[^5]  The hash is a concise way to uniquely represent the file without checking the entire file into source control. Whenever a new external dependency is referenced from a workspace, that dependency」s hash is added to the manifest, either manually or automatically. When Bazel runs a build, it checks the actual hash of its cached dependency against the expected hash defined in the manifest and redownloads the file only if the hash differs.

Bazel和其他一些建立系統透過要求一個覆蓋整個工作區的清單檔案來解決這個問題，該檔案列出了工作區中每個外部依賴項的加密雜湊。每當從工作區引用一個新的外部依賴關係時，該依賴關係的雜湊值就會被手動或自動新增到清單中。Bazel 執行建立時，會將其快取的依賴關係的實際雜湊值與清單中定義的預期雜湊值進行對比，只有在雜湊值不同時才會重新下載檔案。

> [^4]: Such "software supply chain" attacks are becoming more common.
>
> 4   這種『軟體供應鏈』攻擊越來越普遍。
>
> [^5]: Go recently added preliminary support for modules using the exact same system.
>
> 5   Go最近增加了對使用完全相同系統的模組的初步支援。

If the artifact we download has a different hash than the one declared in the manifest, the build will fail unless the hash in the manifest is updated. This can be done automatically, but that change must be approved and checked into source control before the build will accept the new dependency. This means that there」s always a record of when a dependency was updated, and an external dependency can」t change without a corresponding change in the workspace source. It also means that, when checking out an older version of the source code, the build is guaranteed to use the same dependencies that it was using at the point when that version was checked in (or else it will fail if those dependencies are no longer available).

如果我們下載的構件與清單中宣告的雜湊值不同，除非更新清單中的雜湊值，否則建立將失敗。這可以自動完成，但在建立接受新的依賴關係之前，這一變化必須得到批准並檢查到原始碼控制中。這意味著總是有依賴關係更新的記錄，如果工作區原始碼沒有相應的變化，外部依賴關係就不會改變。這也意味著，當簽出一個舊版本的原始碼時，建立保證使用與簽入該版本時相同的依賴關係（否則，如果這些依賴關係不再可用，它將失敗）。

Of course, it can still be a problem if a remote server becomes unavailable or starts serving corrupt data—this can cause all of your builds to begin failing if you don」t have another copy of that dependency available. To avoid this problem, we recommend that, for any nontrivial project, you mirror all of its dependencies onto servers or services that you trust and control. Otherwise you will always be at the mercy of a third party for your build system」s availability, even if the checked-in hashes guarantee its security.

當然，如果一個遠端伺服器變得不可用或開始提供損壞的資料，這仍然是一個問題--如果沒有該依賴項的另一個副本可用，這可能會導致所有建立開始失敗。為了避免這個問題，我們建議，對於任何不重要的專案，你應該把所有的依賴關係映象到你信任和控制的伺服器或服務上。否否則，建立系統的可用性將始終取決於第三方，即使簽入雜湊保證了其安全性。

### Distributed Builds 分散式建立

Google」s codebase is enormous—with more than two billion lines of code, chains of dependencies can become very deep. Even simple binaries at Google often depend on tens of thousands of build targets. At this scale, it」s simply impossible to complete a build in a reasonable amount of time on a single machine: no build system can get around the fundamental laws of physics imposed on a machine」s hardware. The only way to make this work is with a build system that supports distributed builds wherein the units of work being done by the system are spread across an arbitrary and scalable number of machines. Assuming we」ve broken the system」s work into small enough units (more on this later), this would allow us to complete any build of any size as quickly as we」re willing to pay for.

谷歌的程式碼庫非常龐大--有超過20億行的程式碼，依賴關係鏈可以變得非常深。在谷歌，即使是簡單的二進位檔案也常常依賴於成千上萬個建立目標。在這種規模下，要在一台機器上以合理的時間完成建立是根本不可能的：任何建立系統都無法繞過強加給機器硬體的基本物理定律。唯一的辦法是使用支援分散式建立的建立系統，其中系統所完成的工作單元分佈在任意數量且可擴充套件的機器上。假設我們把系統的工作分解成足夠小的單位（後面會有更多介紹），這將使我們能夠以我們可以根據支付的費用來獲得想要的速度完成任何規模的建立。

This scalability is the holy grail we」ve been working toward by defining an artifact-based build system.

透過定義基於構件的建立系統，這種可伸縮性是我們一直致力於實現的法寶。

### Remote caching 遠端快取

The simplest type of distributed build is one that only leverages remote caching, which is shown in Figure 18-2.

最簡單的分散式建立型別是隻利用遠端快取的建立，如圖18-2所示。

![Figure 18-2](./images/Figure%2018-2.jpg)

Figure 18-2. A distributed build showing remote caching

Every system that performs builds, including both developer workstations and continuous integration systems, shares a reference to a common remote cache service. This service might be a fast and local short-term storage system like Redis or a cloud service like Google Cloud Storage. Whenever a user needs to build an artifact, whether directly or as a dependency, the system first checks with the remote cache to see if that artifact already exists there. If so, it can download the artifact instead of building it. If not, the system builds the artifact itself and uploads the result back to the cache. This means that low-level dependencies that don」t change very often can be built once and shared across users rather than having to be rebuilt by each user. At Google, many artifacts are served from a cache rather than built from scratch, vastly reducing the cost of running our build system.

每個執行建立的系統，包括開發人員工作站和連續整合系統，都共享對公共遠端快取服務的引用。這個服務可能是一個高速的本地短期儲存系統，如Redis，或一個雲服務，如谷歌雲儲存。每當使用者需要建立一個構件時，無論是直接建立還是作為一個依賴，系統首先檢查遠端快取，看該構件是否已經存在。如果存在，它可以下載該構件而不是建立它。如果沒有，系統會自己建立構件，並將結果上傳到快取中。這意味著不經常更改的低階依賴項可以建立一次並在使用者之間共享，而不必由每個使用者重新建立。在谷歌，許多構件是從快取中提供的，而不是從頭開始建立的，這大大降低了我們執行建立系統的成本。

For a remote caching system to work, the build system must guarantee that builds are completely reproducible. That is, for any build target, it must be possible to determine the set of inputs to that target such that the same set of inputs will produce exactly the same output on any machine. This is the only way to ensure that the results of downloading an artifact are the same as the results of building it oneself. Fortunately, Bazel provides this guarantee and so supports [remote caching](https://oreil.ly/D9doX). Note that this requires that each artifact in the cache be keyed on both its target and a hash of its inputs—that way, different engineers could make different modifications to the same target at the same time, and the remote cache would store all of the resulting artifacts and serve them appropriately without conflict.

為了使遠端快取系統發揮作用，建立系統必須保證建立是完全可重複的。也就是說，對於任何建立目標，必須能夠確定該目標的輸入集，以便相同的輸入集在任何機器上產生完全相同的輸出。這是確保下載構件的結果與自己建立構件的結果相同的唯一方法。幸運的是，Bazel提供了這種保證，因此支援[遠端快取](https://oreil.ly/D9doX)。請注意，這要求快取中的每個構件都以其目標和輸入的雜湊值為關鍵--這樣，不同的工程師可以在同一時間對同一目標進行不同的修改，而遠端快取將儲存所有結果的構件，並適當地為它們提供服務，而不會產生衝突。

Of course, for there to be any benefit from a remote cache, downloading an artifact needs to be faster than building it. This is not always the case, especially if the cache server is far from the machine doing the build. Google」s network and build system is carefully tuned to be able to quickly share build results. When configuring remote caching in your organization, take care to consider network latencies and perform experiments to ensure that the cache is actually improving performance.

當然，要想從遠端快取中獲得任何好處，下載構件的速度必須比建立它的速度快。但情況並非總是如此，尤其是當快取伺服器遠離進行建立的機器時。谷歌的網路和建立系統是經過精心調整的，能夠快速分享建立結果。在組織中設定遠端快取時，請注意考慮網路延遲，並進行實驗以確保快取實際上正在提高效能

### Remote execution 遠端建立

Remote caching isn」t a true distributed build. If the cache is lost or if you make a low- level change that requires everything to be rebuilt, you still need to perform the entire build locally on your machine. The true goal is to support *remote execution*, in which the actual work of doing the build can be spread across any number of workers. Figure 18-3 depicts a remote execution system.

遠端快取不是真正的分散式建立。如果快取丟失或者進行了需要重建所有內容的低階更改，那麼仍然需要在電腦上本地執行整個建立。遠端快取並不是一個真正的分散式建立。如果快取丟失了，或者如果你做了一個低階別的改變，需要重建所有的東西，你仍然需要在你的機器上執行整個建立。真正的目標是支援*遠端執行*，在這種情況下，進行建立的實際工作可以分散到任何數量的機器上。[圖18-3]（#_bookmark1676）描述了一個遠端執行系統。

![Figure 18-3](./images/Figure%2018-3.png)

Figure 18-3. A remote execution system

The build tool running on each user」s machine (where users are either human engineers or automated build systems) sends requests to a central build master. The build master breaks the requests into their component actions and schedules the execution of those actions over a scalable pool of workers. Each worker performs the actions asked of it with the inputs specified by the user and writes out the resulting artifacts. These artifacts are shared across the other machines executing actions that require them until the final output can be produced and sent to the user.

在每個使用者的機器上執行的建立工具（使用者可以是工程師，也可以是自動建立系統）向中央建立主控器傳送請求。建立主機將請求分解為元件操作，並在可擴充套件的機器資源池上安排這些操作的執行。每個機器根據使用者指定的輸入執行所要求的操作，並寫出結果的構件。這些構件在執行需要它們的操作的其他機器之間共享，直到可以生成最終輸出併發送給使用者。

The trickiest part of implementing such a system is managing the communication between the workers, the master, and the user」s local machine. Workers might depend on intermediate artifacts produced by other workers, and the final output needs to be sent back to the user」s local machine. To do this, we can build on top of the distributed cache described previously by having each worker write its results to and read its dependencies from the cache. The master blocks workers from proceeding until everything they depend on has finished, in which case they」ll be able to read their inputs from the cache. The final product is also cached, allowing the local machine to download it. Note that we also need a separate means of exporting the local changes in the user」s source tree so that workers can apply those changes before building.

實現這樣一個系統最棘手的部分是管理員、主站和使用者的本地機器之間的通訊。某台建立機器可能依賴於其他機器產生的中間構件，而最終輸出需要傳送回用戶的本地機器。要做到這一點，我們可以建立在前面描述的分散式快取之上，讓每個建立機器將其結果寫入快取並從快取中讀取其依賴項。主模組阻止建立程式繼續工作，直到它所依賴的一切完成，在這種情況下，它將能夠從快取中讀取它的輸入。最終的產品也被快取起來，允許本地機器下載它。請注意，我們還需要一種單獨的方法來匯出使用者源樹中的本地更改，以便建立機器可以在建立之前應用這些更改。

For this to work, all of the parts of the artifact-based build systems described earlier need to come together. Build environments must be completely self-describing so that we can spin up workers without human intervention. Build processes themselves must be completely self-contained because each step might be executed on a different machine. Outputs must be completely deterministic so that each worker can trust the results it receives from other workers. Such guarantees are extremely difficult for a task-based system to provide, which makes it nigh-impossible to build a reliable remote execution system on top of one.

要做到這一點，前面描述的基於構件的建立系統的所有部分都需要結合起來。建立環境必須是完全自描述的，這樣我們就可以在沒有人為干預的情況下提高建立的速度。建立過程本身必須是完全自包含的，因為每個步驟可能在不同的機器上執行。輸出必須是完全確定的，這樣每個建立機器就可以相信它從其他建立機器那裡得到的結果。樣的保證對於基於任務的系統來說是非常困難的，這使得在一個系統之上建立一個可靠的遠端執行系統幾乎是不可能的。

**Distributed builds at Google.** Since 2008, Google has been using a distributed build system that employs both remote caching and remote execution, which is illustrated in Figure 18-4.

**谷歌的分散式建立。**自2008年以來，谷歌一直在使用分散式建立系統，該系統同時採用了遠端快取和遠端執行，如[圖18-4]（#_bookmark1678）所示。

![Figure 18-4](./images/Figure%2018-4.png)

*Figure* *18-4. Google」s distributed build system*

Google」s remote cache is called ObjFS. It consists of a backend that stores build outputs in [Bigtables](https://oreil.ly/S_N-D) distributed throughout our fleet of production machines and a frontend FUSE daemon named objfsd that runs on each developer」s machine. The FUSE daemon allows engineers to browse build outputs as if they were normal files stored on the workstation, but with the file content downloaded on-demand only for the few files that are directly requested by the user. Serving file contents on-demand greatly reduces both network and disk usage, and the system is able to [build twice as fast](https://oreil.ly/NZxSp)compared to when we stored all build output on the developer」s local disk.

谷歌的遠端快取系統被稱為ObjFS。它包括一個將建立輸出儲存在[Bigtables](https://oreil.ly/S_N-D)的後端，分佈在我們的生產機群中，以及一個執行在每個開發人員機器上的名為objfsd的前端FUSE守護程式。FUSE守護程序允許工程師像瀏覽儲存在工作站上的普通檔案一樣瀏覽建立輸出，但檔案內容僅針對使用者直接請求的少數檔案按需下載。按需提供檔案內容大大減少了網路和磁碟的使用，系統的建立速度是將所有建立輸出儲存在開發人員的本地磁碟上時的兩倍。

Google」s remote execution system is called Forge. A Forge client in Blaze called the Distributor sends requests for each action to a job running in our datacenters called the Scheduler. The Scheduler maintains a cache of action results, allowing it to return a response immediately if the action has already been created by any other user of the system. If not, it places the action into a queue. A large pool of Executor jobs continually read actions from this queue, execute them, and store the results directly in the ObjFS Bigtables. These results are available to the executors for future actions, or to be downloaded by the end user via objfsd.

谷歌的遠端執行系統被稱為Forge。在Blaze中，一個名為 "Distributor "的Forge用戶端將每個操作的請求傳送到資料中心中名為Scheduler排程器。排程器維護操作結果的快取，允許它在操作已經由系統的任何其他使用者建立時立即回傳回應。如果沒有，它就把操作放到一個佇列中。大量執行器作業從該佇列中連續讀取操作，執行它們，並將結果直接儲存在ObjFS Bigtables中。這些結果可供執行者用於將來的操作，或由最終使用者透過objfsd下載。

The end result is a system that scales to efficiently support all builds performed at Google. And the scale of Google」s builds is truly massive: Google runs millions of builds executing millions of test cases and producing petabytes of build outputs from billions of lines of source code every *day*. Not only does such a system let our engineers build complex codebases quickly, it also allows us to implement a huge number of automated tools and systems that rely on our build. We put many years of effort into developing this system, but nowadays open source tools are readily available such that any organization can implement a similar system. Though it can take time and energy to deploy such a build system, the end result can be truly magical for engineers and is often well worth the effort.

最終的結果是一個可擴充套件的系統，能夠高效地支持在谷歌執行的所有建立。谷歌建立的規模確實是巨大的：谷歌每天執行數以百萬計的建立，執行數以百萬計的測試用例，並從數十億行原始碼中產生數PB的建立輸出。這樣一個系統不僅讓我們的工程師快速建立複雜的程式碼庫，還讓我們能夠實現大量依賴我們建立的自動化工具和系統。我們為開發這個系統付出了多年的努力，但現在開源工具已經很容易獲得，這樣任何組織都可以實現類似的系統。雖然部署這樣一個建立系統可能需要時間和精力，但最終的結果對工程師來說確實是神奇的，而且通常是值得付出努力的。

## Time, Scale, Trade-Offs 時間、規模、權衡

Build systems are all about making code easier to work with at scale and over time. And like everything in software engineering, there are trade-offs in choosing which sort of build system to use. The DIY approach using shell scripts or direct invocations of tools works only for the smallest projects that don」t need to deal with code changing over a long period of time, or for languages like Go that have a built-in build system.

建立系統都是為了使程式碼更易於大規模和長期使用。就像軟體工程一樣，在選擇使用哪種建立系統時也存在權衡。使用shell指令碼或直接呼叫工具的DIY方法只適用於不需要長時間處理程式碼更改的最小專案，或者適用於具有內建建立系統的Go等語言。

Choosing a task-based build system instead of relying on DIY scripts greatly improves your project」s ability to scale, allowing you to automate complex builds and more easily reproduce those builds across machines. The trade-off is that you need to actually start putting some thought into how your build is structured and deal with the overhead of writing build files (though automated tools can often help with this). This trade-off tends to be worth it for most projects, but for particularly trivial projects (e.g., those contained in a single source file), the overhead might not buy you much.

選擇基於任務的建立系統而不是依賴DIY指令碼可以極大地提高專案的可擴充性，允許你自動完成複雜的建立，並更容易在不同的機器上覆制這些建立。權衡之下，你需要真正開始考慮建立是如何構造的，並處理編寫建立檔案的開銷（儘管自動化工具通常可以幫助解決這個問題）。對於大多數專案來說，這種權衡是值得的，但對於特別瑣碎的專案（例如，那些包含在單一源檔案中的專案），開銷可能不會給你帶來太多好處。

Task-based build systems begin to run into some fundamental problems as the project scales further, and these issues can be remedied by using an artifact-based build system instead. Such build systems unlock a whole new level of scale because huge builds can now be distributed across many machines, and thousands of engineers can be more certain that their builds are consistent and reproducible. As with so many other topics in this book, the trade-off here is a lack of flexibility: artifact- based systems don」t let you write generic tasks in a real programming language, but require you to work within the constraints of the system. This is usually not a problem for projects that are designed to work with artifact-based systems from the start, but migration from an existing task-based system can be difficult and is not always worth it if the build isn」t already showing problems in terms of speed or correctness.

隨著專案規模的進一步擴大，基於任務的建立系統開始遇到一些基本問題，而這些問題可以透過使用基於構件的建立系統來彌補。這樣的建立系統開啟了一個全新的規模，因為巨大的建立現在可以分佈在許多機器上，成千上萬的工程師可以更確定他們的建立是一致的和可重複的。就像本書中的許多其他主題一樣，這裡的權衡是缺乏靈活性：基於構件的系統不允許你用真正的程式語言編寫通用任務，而要求你在系統的約束範圍內工作。對於那些從一開始就被設計為與基於構件的系統一起工作的專案來說，這通常不是一個問題，但是從現有的基於任務的系統遷移可能是困難的，而且如果建立在速度或正確性方面還沒有出現問題的話，這並不總是值得的。

Changes to a project」s build system can be expensive, and that cost increases as the project becomes larger. This is why Google believes that almost every new project benefits from incorporating an artifact-based build system like Bazel right from the start. Within Google, essentially all code from tiny experimental projects up to Google Search is built using Blaze.

對一個專案的建立系統進行修改代價耿是昂貴的，而且隨著專案的擴大，成本也會增加。這就是為什麼谷歌認為，幾乎每一個新專案從一開始就可以從Bazel這樣的基於構件的建立系統中獲益。在谷歌內部，從微小的實驗性專案到谷歌搜尋，基本上所有的程式碼都是用Blaze建立的。

## Dealing with Modules and Dependencies 處理模組和依賴關係

Projects that use artifact-based build systems like Bazel are broken into a set of modules, with modules expressing dependencies on one another via BUILD files. Proper organization of these modules and dependencies can have a huge effect on both the performance of the build system and how much work it takes to maintain.

像Bazel這樣使用基於構件的建立系統的專案被分解成一系列模組，模組之間透過BUILD檔案表達彼此的依賴關係。適當地組織這些模組和依賴關係，對建立系統的效能和維護的工作量都有很大的影響。

### Using Fine-Grained Modules and the 1:1:1 Rule 使用細粒度模組和1:1:1規則

The first question that comes up when structuring an artifact-based build is deciding how much functionality an individual module should encompass. In Bazel, a 『module』 is represented by a target specifying a buildable unit like a java_library or a go_binary. At one extreme, the entire project could be contained in a single module by putting one BUILD file at the root and recursively globbing together all of that project」s source files. At the other extreme, nearly every source file could be made into its own module, effectively requiring each file to list in a BUILD file every other file it depends on.

建立基於構件的建立時出現的第一個問題是決定單個模組應該包含多少功能。在Bazel中，一個 "module"是由一個指定可建立單元的目標表示的，如java_library或go_binary。在一個極端，整個專案可以包含在一個單一的module中，方法是把一個BUILD檔案放在根部，然後遞迴地把該專案所有的源檔案放在一起。在另一個極端，幾乎每一個源檔案都可以成為自己的模組，有效地要求每個檔案在BUILD檔案中列出它所依賴的每個其他檔案。

Most projects fall somewhere between these extremes, and the choice involves a trade-off between performance and maintainability. Using a single module for the entire project might mean that you never need to touch the BUILD file except when adding an external dependency, but it means that the build system will always need to build the entire project all at once. This means that it won」t be able to parallelize or distribute parts of the build, nor will it be able to cache parts that it」s already built. One-module-per-file is the opposite: the build system has the maximum flexibility in caching and scheduling steps of the build, but engineers need to expend more effort maintaining lists of dependencies whenever they change which files reference which.

大多數專案都介於這兩個極端之間，這種選擇涉及到效能和可維護性之間的權衡。在整個專案使用一個模組可能意味著除了新增外部依賴項時，你永遠不需要更改建立檔案，但這意味著建立系統將始終需要一次建立整個專案。這意味著它將無法並行化或分發建立的一部分，也無法快取已經建立的部分。每個檔案一個模組的情況正好相反：建立系統在快取和安排建立步驟方面有最大的靈活性，但工程師需要花費更多的精力來維護依賴關係的清單，無論何時他們改變哪個檔案引用哪個檔案。

Though the exact granularity varies by language (and often even within language), Google tends to favor significantly smaller modules than one might typically write in a task-based build system. A typical production binary at Google will likely depend on tens of thousands of targets, and even a moderate-sized team can own several hundred targets within its codebase. For languages like Java that have a strong built- in notion of packaging, each directory usually contains a single package, target, and BUILD file (Pants, another build system based on Blaze, calls this the 1:1:1 rule). Languages with weaker packaging conventions will frequently define multiple targets per BUILD file.

雖然精確的顆粒度因語言而異（甚至在語言內部也是如此），但谷歌傾向於使用比通常在基於任務的建立系統中編寫的模組小得多的模組。在谷歌，一個典型的生產二進位檔案可能會依賴於數以萬計的目標構件，甚至一個中等規模的團隊也可能在其程式碼庫中擁有數百個目標。對於像Java這樣有強大的內建打包概念的語言，每個目錄通常包含一個單獨的套件、目標和BUILD檔案（另一個基於Blaze的建立系統Pants稱之為1:1:1規則）。封裝約定較弱的語言通常會為每個建立檔案定義多個目標。

The benefits of smaller build targets really begin to show at scale because they lead to faster distributed builds and a less frequent need to rebuild targets. The advantages become even more compelling after testing enters the picture, as finer-grained targets mean that the build system can be much smarter about running only a limited subset of tests that could be affected by any given change. Because Google believes in the systemic benefits of using smaller targets, we」ve made some strides in mitigating the downside by investing in tooling to automatically manage BUILD files to avoid burdening developers. Many of these tools are now open source.

較小的建立目標的好處真正開始在規模上表現出來，因為它們可以支援更快的分散式建立和更少的重建目標的需要。當測試進入畫面後，這些優勢變得更加引人注目，因為更細粒度的目標意味著建立系統可以更智慧地只執行可能受任何給定更改影響的有限測試子集。由於谷歌相信使用較小目標的系統性好處，我們透過開發自動管理建立檔案的工具，在減輕不利影響方面取得了一些進展，以避免打擾開發人員。其中許多工具現在都是開源的。

### Minimizing Module Visibility 最小化模組可見性

Bazel and other build systems allow each target to specify a visibility: a property that specifies which other targets may depend on it. Targets can be public, in which case they can be referenced by any other target in the workspace; private, in which case they can be referenced only from within the same BUILD file; or visible to only an explicitly defined list of other targets. A visibility is essentially the opposite of a dependency: if target A wants to depend on target B, target B must make itself visible to target A.

Bazel和其他建立系統允許每個目標指定可見性：一個屬性，指定哪些其他目標可能依賴它。目標可以是公共的，在這種情況下，它們可以被工作區中的任何其他目標引用；private，在這種情況下，它們只能從同一建立檔案中引用；或僅對明確定義的其他目標清單可見。可見性本質上與依賴性相反：如果目標A想要依賴於目標B，目標B必須使自己對目標A可見。

Just like in most programming languages, it is usually best to minimize visibility as much as possible. Generally, teams at Google will make targets public only if those targets represent widely used libraries available to any team at Google. Teams that require others to coordinate with them before using their code will maintain a whitelist of customer targets as their target」s visibility. Each team」s internal implementation targets will be restricted to only directories owned by the team, and most BUILD files will have only one target that isn」t private.

就像在大多數程式語言中，通常最好方法是儘可能地減少可見性。一般來說，谷歌的團隊只有在這些目標代表了谷歌任何團隊都可以使用的廣泛使用的函式庫時，才會將目標公開。要求其他人在使用程式碼之前與他們協調的團隊將保留一份客戶目標白名單，作為其目標的可見性。每個團隊的內部實施目標將被限制在該團隊所擁有的目錄中，而且大多數BUILD檔案將只有一個非私有的目標。

### Managing Dependencies 管理依賴關係

Modules need to be able to refer to one another. The downside of breaking a codebase into fine-grained modules is that you need to manage the dependencies among those modules (though tools can help automate this). Expressing these dependencies usually ends up being the bulk of the content in a BUILD file.

模組需要能夠相互引用。將程式碼庫分解為細粒度模組的缺點是需要管理這些模組之間的依賴關係（儘管工具可以幫助實現自動化）。表達這些依賴關係通常會成為BUILD檔案中的大部分內容。

#### Internal dependencies 內部依賴關係

In a large project broken into fine-grained modules, most dependencies are likely to be internal; that is, on another target defined and built in the same source repository. Internal dependencies differ from external dependencies in that they are built from source rather than downloaded as a prebuilt artifact while running the build. This also means that there」s no notion of 『version』 for internal dependencies—a target and all of its internal dependencies are always built at the same commit/revision in the repository.

在細分為細粒度模組的大型專案中，大多數依賴關係可能是內部的；也就是說，在同一源儲存庫中定義和建立的另一個目標上。內部依賴項與外部依賴項的不同之處在於，它們是從原始碼建立的，而不是在執行建立時作為預建立構件下載的。這也意味著內部依賴項沒有『版本』的概念——目標及其所有內部依賴項始終在儲存庫中的同一送出/修訂中建立。

One issue that should be handled carefully with regard to internal dependencies is how to treat transitive dependencies (Figure 18-5). Suppose target A depends on target B, which depends on a common library target C. Should target A be able to use classes defined in target C?

關於內部依賴關係，應該小心處理的一個問題是如何處理可傳遞依賴關係（圖 18-5）。假設目標A依賴於目標B，而目標B依賴於一個共同的函式庫目標C，那麼目標A是否應該使用目標C中定義的類？

![Figure 18-5](./images/Figure%2018-5.png)

*Figure* *18-5.* *Transitive* *dependencies*

As far as the underlying tools are concerned, there」s no problem with this; both B and C will be linked into target A when it is built, so any symbols defined in C are known to A. Blaze allowed this for many years, but as Google grew, we began to see problems. Suppose that B was refactored such that it no longer needed to depend on C. If B」s dependency on C was then removed, A and any other target that used C via a dependency on B would break. Effectively, a target」s dependencies became part of its public contract and could never be safely changed. This meant that dependencies accumulated over time and builds at Google started to slow down.

就底層工具而言，這沒有問題；B和C在建立目標A時都會連結到目標A中，因此C中定義的任何符號都會被A知道。Blaze允許這一點很多年了，但隨著谷歌的發展，我們開始發現問題。假設B被重構，不再需要依賴C。如果B對C的依賴關係被刪除，A和透過對B的依賴關係使用C的任何其他目標都將中斷。實際上，一個目標的依賴關係成為其公共契約的一部分，永遠無法安全地更改。這意味著依賴性會隨著時間的推移而積累，谷歌的建立速度開始變慢。

Google eventually solved this issue by introducing a 『strict transitive dependency mode』 in Blaze. In this mode, Blaze detects whether a target tries to reference a symbol without depending on it directly and, if so, fails with an error and a shell command that can be used to automatically insert the dependency. Rolling this change out across Google」s entire codebase and refactoring every one of our millions of build targets to explicitly list their dependencies was a multiyear effort, but it was well worth it. Our builds are now much faster given that targets have fewer unnecessary dependencies,[^6] and engineers are empowered to remove dependencies they don」t need without worrying about breaking targets that depend on them.

谷歌最終解決了這個問題，在Blaze中引入了一個 "嚴格傳遞依賴模式"。在這種模式下，Blaze檢測目標是否嘗試引用符號而不直接依賴它，如果是，則失敗，並顯示錯誤和可用於自動插入依賴項的shell指令。在谷歌的整個程式碼庫中推廣這一變化，並重構我們數百萬個建立目標中的每一個，以以明確列出它們的依賴關係，這是一項多年的努力，但這是非常值得的。現在我們的建立速度快多了，因為目標的不必要的依賴性減少了，工程師有權刪除他們不需要的依賴關係，而不用擔心破壞依賴它們的目標。

As usual, enforcing strict transitive dependencies involved a trade-off. It made build files more verbose, as frequently used libraries now need to be listed explicitly in many places rather than pulled in incidentally, and engineers needed to spend more effort adding dependencies to *BUILD* files. We」ve since developed tools that reduce this toil by automatically detecting many missing dependencies and adding them to a *BUILD* files without any developer intervention. But even without such tools, we」ve found the trade-off to be well worth it as the codebase scales: explicitly adding a dependency to *BUILD* file is a one-time cost, but dealing with implicit transitive dependencies can cause ongoing problems as long as the build target exists. [Bazel enforces strict transitive dependencies](https://oreil.ly/Z-CqD)on Java code by default.

像往常一樣，強制執行嚴格的可傳遞依賴關係需要權衡。它使建立檔案更加冗長，因為現在需要在許多地方明確列出常用的函式庫，而不是附帶地將其拉入，而且工程師需要花更多的精力將依賴關係新增到*BUILD*檔案中。我們後來開發了一些工具，透過自動檢測許多缺失的依賴關係並將其新增到*BUILD*檔案中，而不需要任何開發人員的干預，從而減少了這項工作。但即使沒有這樣的工具，我們也發現，隨著程式碼庫的擴充套件，這樣的權衡是非常值得的：明確地在*BUILD*檔案中新增一個依賴關係是一次性的成本，但是只要建立目標存在，處理隱式傳遞依賴項就可能導致持續的問題。Bazel對Java程式碼強制執行嚴格的可傳遞依賴項。

> [^6]: Of course, actually removing these dependencies was a whole separate process. But requiring each target to explicitly declare what it used was a critical first step. See Chapter 22 for more information about how Google makes large-scale changes like this.
>
> 6   當然，實際上刪除這些依賴項是一個完全獨立的過程。但要求每個目標明確宣告它使用了什麼是關鍵的第一步。請參閱第22章，瞭解更多關於谷歌如何做出如此大規模改變的訊息。

### External dependencies 外部依賴

If a dependency isn」t internal, it must be external. External dependencies are those on artifacts that are built and stored outside of the build system. The dependency is imported directly from an *artifact repository* (typically accessed over the internet) and used as-is rather than being built from source. One of the biggest differences between external and internal dependencies is that external dependencies have *versions*, and those versions exist independently of the project」s source code.

如果一個依賴性不是內部的，它一定是外部的。外部依賴關係是指在建立系統之外建立和儲存的構件上的依賴關係。依賴關係直接從*構件庫*（通常透過網際網路訪問）匯入，並按原樣使用，而不是從原始碼建立。外部依賴和內部依賴的最大區別之一是，外部依賴有版本，這些版本獨立於專案的原始碼而存在。

 **Automatic versus manual dependency management.** Build systems can allow the versions of external dependencies to be managed either manually or automatically. When managed manually, the buildfile explicitly lists the version it wants to download from the artifact repository, often using [a semantic version string](https://semver.org/)such as 『1.1.4』. When managed automatically, the source file specifies a range of acceptable versions, and the build system always downloads the latest one. For example, Gradle allows a dependency version to be declared as 『1.+』 to specify that any minor or patch version of a dependency is acceptable so long as the major version is 1.

 **自動與手動依賴管理。**建立系統可以允許手動或自動管理外部依賴的版本。當手動管理時，建立檔案明確列出它要從構件庫中下載的版本，通常使用[語義版本字串](https://semver.org/)，如 "1.1.4"。當自動管理時，源檔案指定了一個可接受的版本範圍，並且建立系統總是下載最新的版本。例如，Gradle允許將依賴版本宣告為 "1.+"，以指定只要主要版本是1，那麼依賴的任何次要或補丁版本都是可以接受的。

Automatically managed dependencies can be convenient for small projects, but they」re usually a recipe for disaster on projects of nontrivial size or that are being worked on by more than one engineer. The problem with automatically managed dependencies is that you have no control over when the version is updated. There」s no way to guarantee that external parties won」t make breaking updates (even when they claim to use semantic versioning), so a build that worked one day might be broken the next with no easy way to detect what changed or to roll it back to a working state. Even if the build doesn」t break, there can be subtle behavior or performance changes that are impossible to track down.

自動管理的依賴關係對於小專案來說是很方便的，但對於規模不小的專案或由多名工程師負責的專案來說，它們通常是帶來災難。自動管理的依賴關係的問題是，你無法控制版本的更新時間。沒有辦法保證外部各方不會進行破壞性的更新（即使他們聲稱使用了語義版本管理），所以前一天還能正常工作的建立，第二天就可能被破壞，而且沒有便捷的方法來檢測什麼變化或將其恢復到工作狀態。即使建立沒有被破壞，也可能有一些細微的行為或效能變化，而這些變化是無法追蹤的。

In contrast, because manually managed dependencies require a change in source control, they can be easily discovered and rolled back, and it」s possible to check out an older version of the repository to build with older dependencies. Bazel requires that versions of all dependencies be specified manually. At even moderate scales, the overhead of manual version management is well worth it for the stability it provides.

相比之下，由於手動管理的依賴項需要在原始碼控制系統中進行更改，它們可以很容易地被發現和回滾，而且有可能檢查出較早版本的儲存庫，用較早的依賴關係進行建立。Bazel要求所有依賴項的版本必須手動指定。即使是中等規模，手動版本管理的開銷對於它提供的穩定性來說也是非常值得的。

**The One-Version Rule.** Different versions of a library are usually represented by different artifacts, so in theory there」s no reason that different versions of the same external dependency couldn」t both be declared in the build system under different names. That way, each target could choose which version of the dependency it wanted to use. Google has found this to cause a lot of problems in practice, so we enforce a strict [*One-Version Rule*](https://oreil.ly/OFa9V)for all third-party dependencies in our internal codebase.

**一個版本的規則。**一個函式庫的不同版本通常由不同的構件來代表，所以在理論上，沒有理由不能在建立系統中以不同的名稱宣告相同外部依賴的不同版本。這樣，每個目標都可以選擇要使用哪個版本的依賴項。谷歌發現這在實踐中會造成很多問題，因此我們在內部程式碼庫中對所有第三方依賴項實施嚴格的一個版本規則。

The biggest problem with allowing multiple versions is the *diamond dependency* issue. Suppose that target A depends on target B and on v1 of an external library. If target B is later refactored to add a dependency on v2 of the same external library, target A will break because it now depends implicitly on two different versions of the same library. Effectively, it」s never safe to add a new dependency from a target to any third-party library with multiple versions, because any of that target」s users could already be depending on a different version. Following the One-Version Rule makes this conflict impossible—if a target adds a dependency on a third-party library, any existing dependencies will already be on that same version, so they can happily coexist.

允許多個版本存在的最主要問題是**鑽石依賴問題**。假設目標A依賴於目標B和外部庫的v1。如果以後對目標B進行重構以新增對同一外部庫的v2的依賴，則目標a將中斷，因為它現在隱式地依賴於同一函式庫的兩個不同版本。實際上，將新的依賴項從目標新增到任何具有多個版本的第三方庫永遠都不安全，因為該目標的任何使用者都可能已經依賴於不同的版本。遵循『一個版本』規則使此衝突不可能發生如果目標在第三方庫上新增依賴項，則任何現有依賴項都將在同一版本上，因此它們可以愉快地共存。

We」ll examine this further in the context of a large monorepo in Chapter 21.

我們將在第21章中結合大型單體的情況進一步研究這個問題。

 **Transitive external dependencies.** Dealing with the transitive dependencies of an external dependency can be particularly difficult. Many artifact repositories such as Maven Central allow artifacts to specify dependencies on particular versions of other artifacts in the repository. Build tools like Maven or Gradle will often recursively download each transitive dependency by default, meaning that adding a single dependency in your project could potentially cause dozens of artifacts to be downloaded in total.

 **可傳遞的外部依賴。**處理外部依賴的可傳遞依賴可能特別困難。許多構件庫（如Maven Central）允許構件指定對倉庫中其他構件的特定版本的依賴性。像Maven或Gradle這樣的建立工具通常會預設遞迴地下載每個橫向依賴，這意味著在你的專案中新增一個依賴可能會導致總共下載幾十個構件。

This is very convenient: when adding a dependency on a new library, it would be a big pain to have to track down each of that library」s transitive dependencies and add them all manually. But there」s also a huge downside: because different libraries can depend on different versions of the same third-party library, this strategy necessarily violates the One-Version Rule and leads to the diamond dependency problem. If your target depends on two external libraries that use different versions of the same dependency, there」s no telling which one you」ll get. This also means that updating an external dependency could cause seemingly unrelated failures throughout the codebase if the new version begins pulling in conflicting versions of some of its dependencies.

這非常方便：在新庫上新增依賴項時，必須跟蹤該函式庫的每個可傳遞依賴項並手動新增它們，這將是一個很大的麻煩。但也有一個巨大的缺點：因為不同的函式庫可能依賴於同一第三方庫的不同版本，所以這種策略必然違反一個版本規則，並導致鑽石依賴問題。如果你的目標依賴於兩個外部庫，而這兩個函式庫使用了同一個依賴的不同版本，那麼就無法預知你最終會得到哪一個版本。這也意味著，如果新版本開始引入某些依賴的衝突版本，更新外部依賴項可能在程式碼庫中引起看似不相關的故障。

For this reason, Bazel does not automatically download transitive dependencies. And, unfortunately, there」s no silver bullet—Bazel」s alternative is to require a global file that lists every single one of the repository」s external dependencies and an explicit version used for that dependency throughout the repository. Fortunately, [Bazel provides tools](https://oreil.ly/kejfX) that are able to automatically generate such a file containing the transitive dependencies of a set of Maven artifacts. This tool can be run once to generate the initial *WORKSPACE* file for a project, and that file can then be manually updated to adjust the versions of each dependency.

因此，Bazel不會自動下載可傳遞依賴。而且，不幸的是，沒有銀彈——Bazel的替代方案是需要一個全域檔案，列出版本庫的每一個外部依賴，以及整個版本庫中用於該依賴的明確版本。幸運的是，[Bazel提供的工具](https://oreil.ly/kejfX)能夠自動生成這樣一個檔案，其中包含一組Maven構件的可傳遞依賴項。該工具可以執行一次，為專案生成初始*WORKSPACE*檔案，然後可以手動更新該檔案，以調整每個依賴的版本。

Yet again, the choice here is one between convenience and scalability. Small projects might prefer not having to worry about managing transitive dependencies themselves and might be able to get away with using automatic transitive dependencies. This strategy becomes less and less appealing as the organization and codebase grows, and conflicts and unexpected results become more and more frequent. At larger scales, the cost of manually managing dependencies is much less than the cost of dealing with issues caused by automatic dependency management.

然而，這裡的權衡是在便捷性和可擴充性之間。小型專案可能更願意不必擔心管理可傳遞依賴本身，並且可能可以不使用自動可傳遞依賴。隨著組織和程式碼庫的增長，這種策略越來越沒有吸引力，衝突和意外結果也越來越頻繁。在更大的範圍內，手動管理依賴關係的成本遠遠低於處理自動依賴關係管理所引起的問題的成本。

**Caching build results using external dependencies.** External dependencies are most often provided by third parties that release stable versions of libraries, perhaps without providing source code. Some organizations might also choose to make some of their own code available as artifacts, allowing other pieces of code to depend on them as third- party rather than internal dependencies. This can theoretically speed up builds if artifacts are slow to build but quick to download.

**使用外部依賴性快取建立結果。**外部依賴最常由發布穩定版本函式庫的第三方提供，可能沒有提供原始碼。一些組織可能也會選擇將他們自己的一些程式碼作為構件來提供，允許其他程式碼作為第三方依賴它們，而不是內部依賴。如果構件的建立速度慢但下載速度快，理論上這可以加快建立速度。

However, this also introduces a lot of overhead and complexity: someone needs to be responsible for building each of those artifacts and uploading them to the artifact repository, and clients need to ensure that they stay up to date with the latest version. Debugging also becomes much more difficult because different parts of the system will have been built from different points in the repository, and there is no longer a consistent view of the source tree.

然而，這也帶來了很多開銷和複雜性：需要有人負責建立每一個構件，並將它們上傳到構件庫，而客戶需要確保它們保持最新的版本。除錯也變得更加困難，因為系統的不同部分將從儲存庫中的不同點建立，並且不再有原始碼樹的一致檢視。

A better way to solve the problem of artifacts taking a long time to build is to use a build system that supports remote caching, as described earlier. Such a build system will save the resulting artifacts from every build to a location that is shared across engineers, so if a developer depends on an artifact that was recently built by someone else, the build system will automatically download it instead of building it. This provides all of the performance benefits of depending directly on artifacts while still ensuring that builds are as consistent as if they were always built from the same source. This is the strategy used internally by Google, and Bazel can be configured to use a remote cache.

解決建立建立時間過長問題的更好方法是使用支援遠端快取的建立系統，如前所述。這樣的建立系統將把每次建立產生的構件儲存到工程師共享的位置，所以如果一個開發者依賴於最近由其他人建立的構件，建立系統將自動下載它而不是建立它。這提供了直接依賴構件的所有效能優勢，同時確保建立的一致性，就像它們總是從同一個源建立一樣。這是谷歌內部使用的策略，Bazel可以設定為使用遠端快取。

**Security and reliability of external dependencies.** Depending on artifacts from third- party sources is inherently risky. There」s an availability risk if the third-party source (e.g., an artifact repository) goes down, because your entire build might grind to a halt if it」s unable to download an external dependency. There」s also a security risk: if the third-party system is compromised by an attacker, the attacker could replace the referenced artifact with one of their own design, allowing them to inject arbitrary code into your build.

**外部依賴的安全性和可靠性。**依賴第三方來源的構件本身是有風險的。如果第三方來源（例如估計函式庫）發生故障，就會有可用性風險，因為如果你無法下載外部依賴，整個建立可能會停止。還有一個安全風險：如果第三方系統被攻擊者破壞了，攻擊者可以用他們自己設計的構件來替換引用的構件，允許他們在你的建立中注入任意程式碼。

Both problems can be mitigated by mirroring any artifacts you depend on onto servers you control and blocking your build system from accessing third-party artifact repositories like Maven Central. The trade-off is that these mirrors take effort and resources to maintain, so the choice of whether to use them often depends on the scale of the project. The security issue can also be completely prevented with little overhead by requiring the hash of each third-party artifact to be specified in the source repository, causing the build to fail if the artifact is tampered with.

這兩個問題都可以透過將你依賴的構件映象到你控制的伺服器上，並阻止你的建立系統訪問第三方構件庫（如Maven Central）來緩解。權衡之下，這些映象需要花費精力和資源來維護，所以是否使用這些映象往往取決於專案的規模。安全問題也可以透過要求在原始碼庫中指定每個第三方構件的雜湊值來完全避免，如果構件被篡改，則會導致建立失敗。

Another alternative that completely sidesteps the issue is to *vendor* your project」s dependencies. When a project vendors its dependencies, it checks them into source control alongside the project」s source code, either as source or as binaries. This effectively means that all of the project」s external dependencies are converted to internal dependencies. Google uses this approach internally, checking every third-party library referenced throughout Google into a *third_party* directory at the root of Google」s source tree. However, this works at Google only because Google」s source control system is custom built to handle an extremely large monorepo, so vendoring might not be an option for other organizations.

另一個完全避開這個問題的辦法是你專案的依賴關係。當專案提供其依賴項時，它會將它們與專案原始碼一起作為原始碼或二進位檔案檢查到原始碼管理中。這實際上意味著該專案所有的外部依賴被轉換為內部依賴。谷歌在內部使用這種方法，將整個谷歌引用的每一個第三方庫檢查到谷歌原始碼樹根部的*third_party*目錄中。然而，這在谷歌是可行的，因為谷歌的原始碼控制系統是定製的，可以處理一個非常大的monorepo，所以對於其他組織來說，vendor可能不是一個選項。

## Conclusion 總結

A build system is one of the most important parts of an engineering organization. Each developer will interact with it potentially dozens or hundreds of times per day, and in many situations, it can be the rate-limiting step in determining their productivity. This means that it」s worth investing time and thought into getting things right.

建立系統是一個工程組織中最重要的部分之一。每個開發人員每天可能要與它互動幾十次或幾百次，在許多情況下，它可能是決定他們生產率的限制性步驟。這意味著，值得花時間和精力把事情做好。

As discussed in this chapter, one of the more surprising lessons that Google has learned is that *limiting engineers」 power and flexibility can improve their productivity*. We were able to develop a build system that meets our needs not by giving engineers free reign in defining how builds are performed, but by developing a highly structured framework that limits individual choice and leaves most interesting decisions in the hands of automated tools. And despite what you might think, engineers don」t resent this: Googlers love that this system mostly works on its own and lets them focus on the interesting parts of writing their applications instead of grappling with build logic. Being able to trust the build is powerful—incremental builds just work, and there is almost never a need to clear build caches or run a 『clean』 step.

正如本章所討論的，谷歌學到的一個更令人驚訝的教訓是，*限制工程師的權力和靈活性可以提高他們的生產力*。我們能夠開發出一個滿足我們需求的建立系統，並不是透過讓工程師自由決定如何進行建立，而是透過開發一個高度結構化的框架，限制個人的選擇，並將最有趣的決策留給自動化工具。不管你怎麼想，工程師們對此並不反感：Googlers喜歡這個系統主要靠自己工作，讓他們專注於編寫應用程式的有趣部分，而不是糾結於建立邏輯。能夠信任建立是一個強大的增量建立，而且幾乎不需要清除建立快取或執行『清理』步驟。

We took this insight and used it to create a whole new type of *artifact-based* build system, contrasting with traditional *task-based* build systems. This reframing of the build as centering around artifacts instead of tasks is what allows our builds to scale to an organization the size of Google. At the extreme end, it allows for a *distributed* *build system* that is able to leverage the resources of an entire compute cluster to accelerate engineers」 productivity. Though your organization might not be large enough to benefit from such an investment, we believe that artifact-based build systems scale down as well as they scale up: even for small projects, build systems like Bazel can bring significant benefits in terms of speed and correctness.

我們接受了這一觀點，並利用它建立了一種全新的基於構件的建立系統，與傳統的建立系統形成對比。這種以構件為中心而不是以任務為中心的建立重構，使我們的建立能夠擴充套件到一個與谷歌規模相當的組織。在極端情況下，它允許一個*分散式建立系統*，能夠利用整個計算叢集的資源來加速工程師的生產力。雖然你的組織可能還沒有大到可以從這樣的投資中獲益，但我們相信，基於構件的建立系統會隨著規模的擴大而縮小：即使對於小型專案，像Bazel這樣的建立系統也可以在速度和正確性方面帶來顯著的好處。

The remainder of this chapter explored how to manage dependencies in an artifact- based world. We came to the conclusion that *fine-grained modules scale better than coarse-grained modules*. We also discussed the difficulties of managing dependency versions, describing the O*ne-Version Rule* and the observation that all dependencies should be *versioned manually and explicitly*. Such practices avoid common pitfalls like the diamond dependency issue and allow a codebase to achieve Google」s scale of billions of lines of code in a single repository with a unified build system.

本章的其餘部分探討瞭如何在一個基於構件的系統中管理依賴關係。我們得出的結論是：*細粒度的模組比粗粒度的模組更容易擴充套件。我們還討論了管理依賴版本的困難，描述了* "一個版本規則 "*，以及所有的依賴都應該*手動和明確的版本*的觀點。這樣的做法可以避免像鑽石依賴問題這樣的常見陷阱，並允許程式碼庫在一個具有統一建立系統的單一儲存庫中實現谷歌數萬億行程式碼的規模。

## TL;DRs  內容提要

- A fully featured build system is necessary to keep developers productive as an organization scales.
- Power and flexibility come at a cost. Restricting the build system appropriately makes it easier on developers.

- 一個功能齊全的建立系統對於保持開發人員在組織規模擴大時的生產力是必要的。
- 權力和靈活性是有代價的。適當地限制建立系統可以使開發人員更容易地使用它。
