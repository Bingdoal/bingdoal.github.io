---
layout: post
title: "Java XML Document 處理大全"
date: 2021-11-26 15:06:24 +0800
category: backend
img: cover/java.jpg
description: 最近的一個案子需要處理大量的 XML 資料，才發現 Java 在這方面的 API 使用起來不是那麼方便，跟 Json 的處理相比大部分都不是很直覺，趁著還有點心得的時候趕緊寫下筆記，期望能將 XML 視為 Json 一樣操作自如
lang: zh-TW
tags: [java, xml]
---

{{page.description}}

## 範例資料
```xml
<?xml version="1.0"?>
<users>
    <user id="1">
        <name>Tony</name>
        <phone>0123456789</phone>
    </user>
    <user id="2">
        <name>Amy</name>
        <phone>9876543210</phone>
    </user>
</users>
```

## 讀取
首先先讀入 XML 資料到 Java 內建函式庫的 `Document` 類別，後續的操作都會針對這個物件展開

```java
String xml = "...";
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = null;
Document doc = null;
try {
    factory.setNamespaceAware(true);
    builder = factory.newDocumentBuilder();
    doc = builder.parse(new InputSource(new StringReader(xml)));
} catch (Exception e) {
    e.printStackTrace();
}
```

### 迴圈讀取
```java
NodeList nodeList = doc.getDocumentElement().getChildNodes();
for (int i = 0; i < nodeList.getLength(); i++) {
    Node node = nodeList.item(i);
    System.out.println("Node name: " + node.getNodeName());
    System.out.println("Node id: " + node.getAttributes().getNamedItem("id").getNodeValue());
    if (node.getNodeType() == Node.ELEMENT_NODE) {
        Element subEle = (Element) node;
        NodeList subList = subEle.getChildNodes();
        for (int j = 0; j < subList.getLength(); j++) {
            Node subNode = subList.item(j);
            System.out.println("Node name: " + subNode.getNodeName());
            System.out.println("Node text: " + subNode.getTextContent());
        }
    }
}
```
因為只有兩層就先寫成兩個迴圈，按照這個邏輯去遞迴的話就可以遍歷整個 XML 的資料樹

### 根據 TagName 讀取
只不過通常的使用情境上都是已經知道內容結構，然後去取得指定的資料，那麼就可以改寫成下面的樣子
```java
NodeList nodeList = doc.getElementsByTagName("user");
for (int i = 0; i < nodeList.getLength(); i++) {
    Node node = nodeList.item(i);
    System.out.println("Node name: " +
                node.getNodeName());
    System.out.println("Node id: " +
                node.getAttributes().getNamedItem("id").getNodeValue());
    if (node.getNodeType() == Node.ELEMENT_NODE) {
        Element subEle = (Element) node;
        System.out.println("name: " +
                    subEle.getElementsByTagName("name").item(0).getTextContent());
        System.out.println("phone: " +
                    subEle.getElementsByTagName("phone").item(0).getTextContent());
    }
}
```

要注意到的是，第一層並不是從 `users` 開始讀，而是直接抓到 `user` 那層，這要特別注意如果其他層有一樣的 tagName 會全部收錄進來，如果設計上沒有考量好有可能會有找錯節點的問題

### XPath
如果說對於結構已經有事先的了解了，其實這個做法是最推薦的

```java
XPath xPath =  XPathFactory.newInstance().newXPath();
String expression = "/users/user";
NodeList nodeList = null;
try {
    nodeList = (NodeList) xPath.compile(expression).evaluate(doc, XPathConstants.NODESET);
    for (int i = 0; i < nodeList.getLength(); i++) {
        Node node = nodeList.item(i);
        System.out.println("Node name: " +
                    node.getNodeName());
        System.out.println("Node id: " +
                    node.getAttributes().getNamedItem("id").getNodeValue());
        if (node.getNodeType() == Node.ELEMENT_NODE) {
            Element subEle = (Element) node;
            System.out.println("name: " +
                        subEle.getElementsByTagName("name").item(0).getTextContent());
            System.out.println("phone: " +
                        subEle.getElementsByTagName("phone").item(0).getTextContent());
        }
    }
} catch (XPathExpressionException e) {
    e.printStackTrace();
}
```
`XPath`是一種 XML 的路徑語言，本來就是用來定位 XML 中的資料位置的，詳細的語法可以[參考](https://zh.wikipedia.org/wiki/XPath)，也可以到[這裡](http://xpather.com/)，實驗看看 `XPath` 的正確性

可以描述完整的路徑去找到需要的節點，比較可以確保不會找錯節點

### toString
有些時候從第三方的函式庫會直接收到 `Document` 的 XML 物件，有些時候需要將其轉換回字串來使用

```java
public String xmlToString(Node doc, Boolean pretty) {
    try {
        StringWriter sw = new StringWriter();
        TransformerFactory tf = TransformerFactory.newInstance();
        Transformer transformer = tf.newTransformer();
        transformer.setOutputProperty(OutputKeys.OMIT_XML_DECLARATION, "yes");
        transformer.setOutputProperty(OutputKeys.METHOD, "xml");
        transformer.setOutputProperty(OutputKeys.ENCODING, "UTF-8");
        if (pretty) {
            transformer.setOutputProperty(OutputKeys.INDENT, "yes");
        }
        transformer.transform(new DOMSource(doc), new StreamResult(sw));
        return sw.toString();
    } catch (Exception ex) {
        ex.printStackTrace()
    }
}
```

如果需要轉換成 Json 或是其他格式，可以參考上面遍歷整個 XML 的方法，再根據屬性去建立整個結構

## 建立
有時候會需要從其他格式轉換到 XML 來，需要從頭建立整個 `Document` 的結構，一樣用上面的結構舉例來從頭建立

```java
DocumentBuilderFactory docFactory = DocumentBuilderFactory.newInstance();
DocumentBuilder docBuilder = docFactory.newDocumentBuilder();
Document doc = docBuilder.newDocument();

Element usersEle = doc.createElement("users");

Element userTony = doc.createElement("user");
Element tonyName = doc.createElement("name");
Element tonyPhone = doc.createElement("phone");
userTony.setAttribute("id", "1");
tonyName.setTextContent("Tony");
tonyPhone.setTextContent("0123456789");
userTony.appendChild(tonyName);
userTony.appendChild(tonyPhone);
usersEle.appendChild(userTony);

Element userAmy = doc.createElement("user");
Element amyName = doc.createElement("name");
Element amyPhone = doc.createElement("phone");
userAmy.setAttribute("id", "2");
amyName.setTextContent("Amy");
amyPhone.setTextContent("9876543210");
userAmy.appendChild(amyName);
userAmy.appendChild(amyPhone);
usersEle.appendChild(userAmy);

doc.appendChild(usersEle);

System.out.println(xmlToString(doc, true));
```

## 刪除
刪除 XML 的節點在內建的 API 中只有提供

```java
public Node removeChild(Node oldChild) throws DOMException;
```

注意到方法的傳入參數是 `Node` 也就是說，在刪除之前必須先定位到想刪除的節點，因此操作上可能會像這樣

```java
Node node = doc.getElementsByTagName("user").item(0);
doc.removeChild(node);
```

---

## 結語
整理過後會發現也沒這麼複雜，只是跟 Json 的操作比起來還是繁雜了一點，被各種方便的函式庫寵壞了