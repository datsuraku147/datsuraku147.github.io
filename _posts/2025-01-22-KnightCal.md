---
title: KnightCal
date: 2025-01-22 18:48:20 +0800
categories: [CTF]
tags: [web]     # TAG names should always be lowercase
description: My writeup for the KnightCTF 2025 "KnightCal" challenge
---

## Introduction

Hi! For my first write-up, I decided to write about something easy. Enjoy :D


## First Glance

The target site is pretty simple, we can enter numbers and math expressions then it will calculate the result for us hence the name "KnightCal"


![Figure 1](https://github.com/user-attachments/assets/e5a2f685-6f7f-46cf-b818-81b5c188b3b3){: width="700" height="400" }

&emsp; &emsp; &emsp; ***Figure 1***



## Unnecessary Feature???

If you look at the results it's giving us, we get additional information **"File already exists with name: d.txt"**

After playing with it I concluded that each number represents a certain letter. For example in ***Figure 1*** `2 == d.txt`
When we enter `22,` the result is `dd.txt.` This means that it concatenates the string based on what the number represents.

## Numbers --> TXT --> flag.txt

The next thing I did was to get all of the letters that the numbers `1-9` represented.

```
1 = l.txt
2 = d.txt
3 = b.txt 
4 = h.txt
5 = g.txt
6 = c.txt
7 = f.txt
8 = e.txt
9 = a.txt
```

and since this is a CTF the first thing that came to my mind is to enter the numbers that will result in `flag.txt`

`7195 == flag.txt`

And that's how we get the flag :P

![Figure 2](https://github.com/user-attachments/assets/39950f84-7287-4f68-ab43-0a08ac502034){: width="700" height="400" }

&emsp; &emsp; &emsp; ***Figure 2***

That's all for this writeup. Thanks for reading and have a nice day :D

***-Datsuraku147***