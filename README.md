# 使用 jsoup 解析 HTML

[![宣传图片](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/)

本指南将向你介绍如何在 Java 中使用 `jsoup` 来解析 HTML。你将学习如何使用 DOM 方法、处理分页，以及如何优化你的网页解析流程。

- [使用 DOM 方法与 Jsoup](#使用-dom-方法与-jsoup)
  - [getElementById](#getelementbyid)
  - [getElementsByTag](#getelementsbytag)
  - [getElementsByClass](#getelementsbyclass)
  - [getElementsByAttribute](#getelementsbyattribute)
- [进阶技巧](#进阶技巧)
  - [CSS 选择器](#css-选择器)
  - [处理分页](#处理分页)
- [综合示例](#综合示例)

## 开始使用

本教程假设你使用 [Maven](https://maven.apache.org/) 进行依赖管理。

在安装 Maven 后，先创建一个名为 `jsoup-scraper` 的新 Java 项目：

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=jsoup-scraper -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

要添加所需的依赖，请将 `pom.xml` 中的内容替换为以下代码：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>jsoup-scraper</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>jsoup-scraper</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.jsoup</groupId>
        <artifactId>jsoup</artifactId>
        <version>1.16.1</version>
    </dependency>
  </dependencies>
  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
</properties>
</project>
```

然后在 `App.java` 文件中粘贴以下代码：

```java
package com.example;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

public class App {
    public static void main(String[] args) {

        String url = "https://books.toscrape.com";
        int pageCount = 1;

        while (pageCount <= 1) {

            try {
                System.out.println("---------------------PAGE "+pageCount+"--------------------------");

                //连接到网站并获取其 HTML
                Document doc = Jsoup.connect(url).get();
            
                //打印标题
                System.out.println("Page Title: " + doc.title());
            
                
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        System.out.println("Total pages scraped: "+(pageCount-1));
    }
}
```

- `Jsoup.connect("https://books.toscrape.com").get()`：此行将抓取页面并返回一个可操作的 `Document` 对象。  
- `doc.title()` 返回该 HTML 文档的标题，这里为：`All products | Books to Scrape - Sandbox`。

## 使用 DOM 方法与 Jsoup

`jsoup` 提供了多种方法来在 DOM (文档对象模型) 中寻找元素。以下方法都可用于轻松定位网页中的元素。

- `getElementById()`：根据元素的 `id` 查找。  
- `getElementsByClass()`：根据元素的 CSS class 查找所有匹配元素。  
- `getElementsByTag()`：根据元素的 HTML 标签查找所有匹配元素。  
- `getElementsByAttribute()`：根据元素包含的某个属性来查找所有匹配元素。

### getElementById

在我们要爬取的网站中，侧边栏包含一个 `div`，其 `id` 为 `promotions_left`：

![查看侧边栏元素](https://github.com/bright-cn/jsoup-html-parsing/blob/main/Images/Inspect-the-sidebar.png)

```java
//根据 ID 查找
Element sidebar = doc.getElementById("promotions_left");

System.out.println("Sidebar: " + sidebar);
```

此代码将输出你在浏览器检查工具中看到的 HTML 元素：

```
Sidebar: <div id="promotions_left">
</div>
```

### getElementsByTag

`getElementsByTag()` 用于找到页面中所有某个标签的元素。在本页面，每本书都位于一个单独的 `article` 标签中：

![查看图书元素](https://github.com/bright-cn/jsoup-html-parsing/blob/main/Images/Inspect-books.png)

下面的代码将返回一个包含所有书籍信息的数组，这也是后续获取更多数据的基础：

```java
//根据标签查找
Elements books = doc.getElementsByTag("article");
```

### getElementsByClass

让我们看看书价相关的元素。价格所在的 class 为 `price_color`：

![查看价格元素](https://github.com/bright-cn/jsoup-html-parsing/blob/main/Images/Inspect-price.png)

下面的代码片段会查找所有 `price_color` class 的元素，并使用 `.first().text()` 打印第一个的文字内容：

```java
System.out.println("Price: " + book.getElementsByClass("price_color").first().text());
```

### getElementsByAttribute

下面是用 `getElementsByAttribute("href")` 查找所有包含 `href` 属性的元素：

```java
//根据属性查找
Elements hrefs = book.getElementsByAttribute("href");
System.out.println("Link: https://books.toscrape.com/" + hrefs.first().attr("href"));
```

## 进阶技巧

### CSS 选择器

如果需要使用多种条件来定位元素，可以将 CSS 选择器传给 `select()` 方法。它将返回所有符合选择器的对象数组。下面的示例中，我们使用 `li[class='next']` 来查找所有具有 `next` class 的 `li` 元素：

```java
Elements nextPage = doc.select("li[class='next']");
```

### 处理分页

在处理分页时，我们先用 `nextPage.first()` 获取数组中的第一个元素，然后调用 `getElementsByAttribute("href").attr("href")` 获取其 `href` 值。

由于在翻页到第 2 页后，链接中会去掉 `catalogue`，我们检查并手动添加它。然后将这个更新后的链接与基础 URL 组合，得到下一页的完整链接。

```java
if (!nextPage.isEmpty()) {
    String nextUrl = nextPage.first().getElementsByAttribute("href").attr("href");
    if (!nextUrl.contains("catalogue")) {
        nextUrl = "catalogue/"+nextUrl;
    } 
    url = "https://books.toscrape.com/" + nextUrl;
    pageCount++;
}
```

## 综合示例

以下是完整的 Java 代码。若想爬取多页数据，只需将 `while (pageCount <= 1)` 中的 `1` 改大一些，例如想爬取 4 页，就写成 `while (pageCount <= 4)`。

```java
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

public class App {
    public static void main(String[] args) {

        String url = "https://books.toscrape.com";
        int pageCount = 1;

        while (pageCount <= 1) {

            try {
                System.out.println("---------------------PAGE "+pageCount+"--------------------------");

                //连接到网站并获取其 HTML
                Document doc = Jsoup.connect(url).get();
            
                //打印标题
                System.out.println("Page Title: " + doc.title());
            
                //根据 ID 查找
                Element sidebar = doc.getElementById("promotions_left");

                System.out.println("Sidebar: " + sidebar);

                //根据标签查找
                Elements books = doc.getElementsByTag("article");

                for (Element book : books) {
                    System.out.println("------Book------");
                    System.out.println("Title: " + book.getElementsByTag("img").first().attr("alt"));
                    System.out.println("Price: " + book.getElementsByClass("price_color").first().text());
                    System.out.println("Availability: " + book.getElementsByClass("instock availability").first().text());

                    //根据属性查找
                    Elements hrefs = book.getElementsByAttribute("href");
                    System.out.println("Link: https://books.toscrape.com/" + hrefs.first().attr("href"));
                }

                //使用 CSS 选择器找到下一页的按钮
                Elements nextPage = doc.select("li[class='next']");
                if (!nextPage.isEmpty()) {
                    String nextUrl = nextPage.first().getElementsByAttribute("href").attr("href");
                    if (!nextUrl.contains("catalogue")) {
                        nextUrl = "catalogue/"+nextUrl;
                    } 
                    url = "https://books.toscrape.com/" + nextUrl;
                    pageCount++;
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        System.out.println("Total pages scraped: "+(pageCount-1));
    }
}
```

编译代码：

```bash
mvn package
```

然后执行：

```bash
mvn exec:java -Dexec.mainClass="com.example.App"
```

下面是从第一页获取到的输出示例：

```
---------------------PAGE 1--------------------------
Page Title: All products | Books to Scrape - Sandbox
Sidebar: <div id="promotions_left">
</div>
------Book------
Title: A Light in the Attic
Price: £51.77
Availability: In stock
Link: https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html
------Book------
Title: Tipping the Velvet
Price: £53.74
Availability: In stock
Link: https://books.toscrape.com/catalogue/tipping-the-velvet_999/index.html
------Book------
Title: Soumission
Price: £50.10
Availability: In stock
Link: https://books.toscrape.com/catalogue/soumission_998/index.html
------Book------
Title: Sharp Objects
Price: £47.82
Availability: In stock
Link: https://books.toscrape.com/catalogue/sharp-objects_997/index.html
------Book------
Title: Sapiens: A Brief History of Humankind
Price: £54.23
Availability: In stock
Link: https://books.toscrape.com/catalogue/sapiens-a-brief-history-of-humankind_996/index.html
------Book------
Title: The Requiem Red
Price: £22.65
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-requiem-red_995/index.html
------Book------
Title: The Dirty Little Secrets of Getting Your Dream Job
Price: £33.34
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-dirty-little-secrets-of-getting-your-dream-job_994/index.html
------Book------
Title: The Coming Woman: A Novel Based on the Life of the Infamous Feminist, Victoria Woodhull
Price: £17.93
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-coming-woman-a-novel-based-on-the-life-of-the-infamous-feminist-victoria-woodhull_993/index.html
------Book------
Title: The Boys in the Boat: Nine Americans and Their Epic Quest for Gold at the 1936 Berlin Olympics
Price: £22.60
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-boys-in-the-boat-nine-americans-and-their-epic-quest-for-gold-at-the-1936-berlin-olympics_992/index.html
------Book------
Title: The Black Maria
Price: £52.15
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-black-maria_991/index.html
------Book------
Title: Starving Hearts (Triangular Trade Trilogy, #1)
Price: £13.99
Availability: In stock
Link: https://books.toscrape.com/catalogue/starving-hearts-triangular-trade-trilogy-1_990/index.html
------Book------
Title: Shakespeare's Sonnets
Price: £20.66
Availability: In stock
Link: https://books.toscrape.com/catalogue/shakespeares-sonnets_989/index.html
------Book------
Title: Set Me Free
Price: £17.46
Availability: In stock
Link: https://books.toscrape.com/catalogue/set-me-free_988/index.html
------Book------
Title: Scott Pilgrim's Precious Little Life (Scott Pilgrim #1)
Price: £52.29
Availability: In stock
Link: https://books.toscrape.com/catalogue/scott-pilgrims-precious-little-life-scott-pilgrim-1_987/index.html
------Book------
Title: Rip it Up and Start Again
Price: £35.02
Availability: In stock
Link: https://books.toscrape.com/catalogue/rip-it-up-and-start-again_986/index.html
------Book------
Title: Our Band Could Be Your Life: Scenes from the American Indie Underground, 1981-1991
Price: £57.25
Availability: In stock
Link: https://books.toscrape.com/catalogue/our-band-could-be-your-life-scenes-from-the-american-indie-underground-1981-1991_985/index.html
------Book------
Title: Olio
Price: £23.88
Availability: In stock
Link: https://books.toscrape.com/catalogue/olio_984/index.html
------Book------
Title: Mesaerion: The Best Science Fiction Stories 1800-1849
Price: £37.59
Availability: In stock
Link: https://books.toscrape.com/catalogue/mesaerion-the-best-science-fiction-stories-1800-1849_983/index.html
------Book------
Title: Libertarianism for Beginners
Price: £51.33
Availability: In stock
Link: https://books.toscrape.com/catalogue/libertarianism-for-beginners_982/index.html
------Book------
Title: It's Only the Himalayas
Price: £45.17
Availability: In stock
Link: https://books.toscrape.com/catalogue/its-only-the-himalayas_981/index.html
Total pages scraped: 1
```

## 结论

对动态网站(如商品列表、新闻、研究数据等)进行爬取可能具有一定挑战。[Bright Data 的多种工具](https://www.bright.cn/products)可以帮助你更好地扩展爬取工作：

- [住宅代理](https://www.bright.cn/proxy-types/residential-proxies)：避开 IP 封禁和地理限制。  
- [抓取浏览器](https://www.bright.cn/products/scraping-browser)：轻松处理大量 JavaScript 页面。  
- [可直接使用的数据集](https://www.bright.cn/products/datasets)：无需自行爬取即可获取结构化数据。

将这些与 jsoup 结合使用，可在高效率、低风险的前提下进行数据提取。立即免费试用吧！
