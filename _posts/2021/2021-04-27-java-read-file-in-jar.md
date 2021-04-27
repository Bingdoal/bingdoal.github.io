---
layout: post
title: "[踩雷紀錄] 在 build 成 jar 的 Java 中讀在 jar 中的檔案: URI is not hierarchical"
date: 2021-04-27 09:45:01 +0800
category: others
img: cover/java.jpg
description: 標題看起來有點繞口，不過這個問題倒真的令人困擾，主要困擾在於開發時完全沒有問題，但是到了 build 成 jar 之後，狀況才會出現，也造成要除錯十分麻煩，必須改完 code 然後 build 成 jar 才能確定問題，也沒辦法用 debugger
lang: zh-TW
tags: [java, 踩雷紀錄]
---

{{page.description}}

# 問題描述

+ 問題主要發生於下面程式碼:

```java
URL url = this.getClass().getResource("file/");
if (url == null) {
    url = this.getClass().getClassLoader().getResource("file/");
}
File sourceDir = new File(url.toURI());
List<File> files = Files.walk(sourceDir.toPath())
            .filter(p -> p.getFileName().toString().endsWith(".csv"))
            .map(Path::toFile).collect(Collectors.toList());
```

主要需求為讀取專案 `src/main/resources/file/` 底下的所有 `.csv` 檔，以上程式碼在開發階段都沒有問題，但一旦 build 成 jar 就會吐錯誤 `URI is not hierarchical` 發生在 `url.toURI()`


查了幾篇網路文章有講到讀取 jar 中的 resources 不能用 `getResource()` 要用 `getResourceAsStream()` 的方式才能讀到檔案，測試過後的確可以使用，但 `getResourceAsStream()` 不能讀取 jar 中的目錄結構，對於我的需求是讀取整個目錄下的內容會有點問題，以此為出發點發展出下面解方

```java
List<File> files = new ArrayList<>();
URL jar = this.getClass().getProtectionDomain().getCodeSource().getLocation();
ZipInputStream zip = new ZipInputStream(jar.openStream());
ZipEntry ze;
while ((ze = zip.getNextEntry()) != null) {
    String entryName = ze.getName();
    if (entryName.endsWith(".csv")) {
        String path = jar + entryName.split("classes/")[1];
        InputStream inputStream = this.getClass().getResourceAsStream(path);
        if (inputStream == null) {
            inputStream = this.getClass().getClassLoader().getResourceAsStream(path);
        }
        File file = File.createTempFile("temp", ".csv");
        try (FileOutputStream out = new FileOutputStream(file)) {
            IOUtils.copy(inputStream, out);
        }
        files.add(file);
        inputStream.close();
    }
}
```

有點土法煉鋼的方式，從上面得知 `getResourceAsStream()` 可以讀取 jar 中的檔案所以重點在於如何定位到裡面的檔案

因此用 `this.getClass().getProtectionDomain().getCodeSource().getLocation()` 去取得 jar 的位置，然後用 `ZipInputStream` 拿到裡面的目錄結構，再將 jar 的位置以及目錄結構組合成完整路徑，透過 `getResourceAsStream()` 拿到檔案的 `InputStream` 最後根據需求轉換成 `File`，這才成功解掉這個問題