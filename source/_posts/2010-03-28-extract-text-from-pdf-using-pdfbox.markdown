---
layout: post
title: "使用pdfbox 抽取PDF文件中的文本"
date: 2010-03-28 16:41
comments: true
categories: tools
---
``` java
package com.yanjiuyanjiu.search;

import java.io.File;
import java.io.IOException;

import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.util.PDFTextStripper;

/**
 * 抽去PDF文件中的文本.
 * @author soulmachine
 *
 */
public final class PDFboxTest {
 /** 禁止创建对象. */
 private PDFboxTest() {
 }
 /**
 * 抽取PDF中的文本.
 * @param f PDF文件
 * @return PDF对应的文本字符串
 */
 public static String getText(final File f) {
 String text = "";
 try {
 PDDocument pdfdocument = PDDocument.load(f);
 PDFTextStripper stripper = new PDFTextStripper();
 stripper.setStartPage(1); // 只抽取第1页和第2页
 stripper.setEndPage(2);
 text = stripper.getText(pdfdocument);
 pdfdocument.close();

 } catch (IOException e) {
 e.printStackTrace();
 }

 return text;
 }

 /** 测试.
 *
 * @param args PDF文件路径
 */
 public static void main(final String[]  args) {
 File file = new File(args[0]);
 System.out.println(PDFboxTest.getText(file));
 }
}
```