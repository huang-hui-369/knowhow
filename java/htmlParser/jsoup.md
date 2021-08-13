# 概述
jsopu 是html parser的一种，特征就是支持通过id，class等css selector来检索。

# maven 导入
```
<!-- https://mvnrepository.com/artifact/org.jsoup/jsoup -->
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.13.1</version>
</dependency>

```

# 通过url来解析html

``` java
package sample.jsoup;

import java.io.IOException;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

public class Main {

    public static void main(String[] args) throws IOException {
        Document document = Jsoup.connect("https://www.google.com/search")
                                 .data("q", "jsoup")
                                 .get();

        Elements elements = document.select(".tF2Cxc");

        for (Element element : elements) {
            System.out.println(element.text());
        }
    }
}
```

# 通过html字符串来解析html
```java
String source = "<p>都民の皆様へ</p><p><br/></p><p>　現在、都では、「新型コロナウイルス感染拡大防止のための東京都における緊急事態措置等」を取りまとめ、人流の抑制を最優先に、不要不急の外出自粛、催物（イベント等）の開催制限等を要請しています。</p><p><br/></p><p>感染拡大防止のため、更なるご協力をよろしくお願いいたします。</p><p><br/></p><p>test<a href=\"https://www.bousai.metro.tokyo.lg.jp/1007617/1014242.htm\" target=\"_blank\">あいうえおURL</a>テスト<br/></p>";
	    Document doc = Jsoup.parse(source);
        // select出所有的p标签
	    Elements pList = doc.select("p");
	    for (Element p: pList){
            // select出所有的a标签
	    	Elements aList = p.select("a");
	    	if(aList!=null && aList.size()>0) {
                // text()<p></p>之间的text内容
	    		String pstr = p.text();
	    		for(Element a : aList) {
                    // attr() a标签的href属性的内容
	    			pstr = pstr.replaceFirst(a.text(), "(" + a.attr("href") + ")");
	    		}
	    		System.out.println(pstr);
	    	} else {
	    		System.out.println(p.text());
	    	}
	    }
```

# tag检索

```java
package sample.jsoup;

import java.io.File;
import java.io.IOException;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;

public class Main {

    public static void main(String[] args) throws IOException {
        Document document = Jsoup.parse(new File("input.html"), "UTF-8");
        // 通过id检索    
        Element element = document.getElementById("hoge");
        System.out.println(element.outerHtml());
    }
}
```
# css selector 检索

```java
package sample.jsoup;

import java.io.File;
import java.io.IOException;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

public class Main {

    public static void main(String[] args) throws IOException {
        Document document = Jsoup.parse(new File("input.html"), "UTF-8");
        // 检索id为hogo下的 ul 的class为error的元素    
        Elements elements = document.select("#hoge ul .error");
        for (Element element : elements) {
            System.out.println(element.outerHtml());
        }
    }
}
```

# method
- html() メソッドで、そのタグの中身を HTML 形式の文字列で取得できる。
- text() メソッドで、そのタグの中身のうち、テキスト部分だけを抽出した文字列を取得できる。
- outerHtml() メソッドで、そのタグ自身を HTML 形式の文字列で取得できる。