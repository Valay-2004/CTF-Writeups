## Task 1 - OSINT time

It's just another Monday morning on your mail delivery route when an unusual letter catches your eye. The envelope is battered, riddled with holes as if it's been through a storm. The address is barely legible, and your coworkers at the post office wave it off as a lost cause.

But something about it nags at you.

You carefully open the damp envelope. Inside, you find a faded newspaper clipping and a short handwritten note. The clipping is torn and water-damaged, with key sections missing. The note is personal, clearly not meant for your eyes, but the fragments you can read hint at a story buried in time.

**Objective:**  Use the clues provided in the zip file to uncover the full name and age of the person mentioned in the note.

Flag format: {Name_Surname_age}, only the first letter of the name and surname should be capitalised

Example: {Pierre-Henry_Lagaffe_23}

---

We're three files (2 images + 1 note).

1. Letter
   ![Letter](images/letter.png)

2. NewsPaper clipping
   ![NewsPaper clipping](images/Newspaper_clipping.png)
3. Note

```
Mon cher Édouard,

Aujourd'hui, en rangeant le grenier chez mes grands-parents, je suis tombée sur cette vieille coupure de journal. Ton arrière-grand-père n'avait même pas l'âge de passer le permis quand il s'est distingué ce jour-là. Le benjamin de l'équipe, et certainement pas le moins courageux.

Il serait si fier de te voir sur l'eau à ton tour.

Avec toute mon affection,
Audette
```

---

And we've to solve these two questions:

1. What is the postal code of the delivery address on the envelope?
2. What is the flag?

---

Okay, so after some digging and searching I found out that we can Identify the postal code of the delivery by decoding the `French Postal Barcode` printing on the bottom right corner of the envelope

I then tried to decode it using [dcode.fr/french-postal-barcode](https://www.dcode.fr/french-postal-barcode) decoder and we found these results:

|     |       |
| --- | ----- |
| ↤   | 29760 |
| ↦   | 06792 |

![](attachment/24bd29ce651aacf4c8be67f343ff06f9.excalidraw)

So our answer for the first question about the postal code is `29760` which belongs to the ==Finistère department in Brittany, northwestern France==, primarily covering the commune of **[Taulé](https://www.google.com/search?q=Taul%C3%A9&client=firefox-b-d&hs=kKjp&sca_esv=53030c98d15754b0&biw=1536&bih=731&sxsrf=ANbL-n7bVZUy0UX0iPRAr5vdZjyjy96iAg%3A1776504801911&ei=4U_jaZKiN9XBvr0PtafD6Qg&ved=2ahUKEwj4jaGOjfeTAxWLmq8BHTDZAD8QgK4QegQIARAC&uact=5&oq=france+29670&gs_lp=Egxnd3Mtd2l6LXNlcnAiDGZyYW5jZSAyOTY3MDIGEAAYFhgeMgYQABgWGB4yCxAAGIAEGIYDGIoFMgsQABiABBiGAxiKBTILEAAYgAQYhgMYigUyCBAAGIAEGKIEMgUQABjvBTIIEAAYgAQYogQyBRAAGO8FSOopUOcFWPMjcAF4AZABAZgB5AagAdQnqgELMi0xLjMuMi4yLjK4AQPIAQD4AQGYAgqgAs0hwgIKEAAYsAMY1gQYR8ICCxAAGIAEGJECGIoFwgIKEAAYgAQYQxiKBcICDRAAGIAEGLEDGEMYigXCAg0QLhiABBixAxhDGIoFwgIKEC4YgAQYQxiKBcICBRAAGIAEwgIOEC4YgAQYsQMY0QMYxwHCAggQABiABBixA8ICBRAuGIAEmAMAiAYBkAYIkgcNMS4wLjEuMy4zLjEuMaAH1kmyBwsyLTEuMy4zLjEuMbgHxCHCBwQyLTEwyAcpgAgA&sclient=gws-wiz-serp&mstk=AUtExfAGi4OeEZLafFQ-jyPeTqLgUzrt_kiMwq7if_SLPvcqtvNmyomGoEA6w1IubXt3vvveQ8tJvFjkcx-fRcHy941IpOFiM9427vSXpZY4PneFAEPH8f5wbbfgPwM49ihaUAQp0eoL0hTF-hH5DxxLNGixb_l5G63mOGz8NzCWR_2ZIjU1T0QDe2d6qZm1lk5xZrTOeJMDC8DWPfxIW6AIQF0qZw&csui=3)**

1. What is the postal code of the delivery address on the envelope?
   --> `29760`

---

### What the note says:

> [!Letter]
> My dear Édouard,  
> Today, while cleaning out the attic at my grandparents', I came across this old newspaper clipping. Your great-grandfather wasn't even old enough to get his driver's license when he distinguished himself that day. The youngest member of the team/crew, and certainly not the least courageous.  
> He would be so proud to see you on the water in your turn.  
> With all my affection,  
> Audette

> [!tip]
> **The newspaper clipping and the personal note are directly connected to a specific maritime catastrophe in Brittany on 23 May 1925.**

> This was the tragic “`Naufrage de Penmarc’h`” (or “Catastrophe du 23 `mai` 1925” in `Penmarc’h`, `Finistère`, western Brittany). A sudden violent storm struck while local fishing boats were at sea. Two fishing vessels capsized first. Then the two lifeboats (`canots` de `sauvetage`) from `Kérity` and `Saint-Pierre-Penmarc’h` rushed to rescue them — and they also capsized in the heavy seas. In total, **27 men died**: 12 fishermen + 15 rescuers/`sauveteurs`. It left 23 widows and 45 orphans and was one of the worst local maritime disasters of the era. The event made national news and was heavily covered in regional papers like _`L’Ouest-Éclair`_.

Check this article which will give you the answer for the youngest rescuer [kbcpenmarch](https://kbcpenmarch.franceserv.com/la-catastrophe-du-23-mai-1925-selon-les-annales-du-sauvetage.html) who was

```
2nd Class Silver Medal.
GOURLAOUEN (Yves-Marie) says Yvon, 15 years old, foam.
```

![Image](https://kbcpenmarch.franceserv.com/_media/img/medium/depute-danielou-entoure-de-sauveteurs-du-sinistre-du-23-mai-1925.webp)

> May 28, 1925 in the shelter of the Kérity lifeboat:  
>  The Deputy Danielou surrounded by the rescuers of May 23,

_Image is not mine I have only added here for the reference You can see the image in the article_

And thus we got the great-grandparent of `Édouard` to which the letter was being delivered
So our final flag would be
Ans --> `THM{Yves-Marie_Gourlaouen_15}`

2. What is the flag?
   --> `THM{Yves-Marie_Gourlaouen_15}`

---
