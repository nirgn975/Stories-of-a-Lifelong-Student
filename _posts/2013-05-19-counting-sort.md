---
title: 'מיון מניה (Counting Sort)'
author: nirgn
layout: post
summary: "מיון מניה, מיועד כדי למיין מספרים שלמים בטווח חסום וידוע מראש (כלומר המיון מתבסס על ידע קודם שניתן לנו על הקלט)."
category: Algorithms
---
Counting Sort - מיון מניה, מיועד כדי למיין מספרים שלמים בטווח חסום וידוע מראש (כלומר המיון מתבסס על ידע קודם שניתן לנו על הקלט).

<!--more-->

<div class="left">
  <img src="/assets/img/posts/counting-sort/counting-sort-animation.gif" alt="Counting Sort Animation">
</div>

בהתחשב בידע זה, אנו יודעים שמערך הקלט מכיל מספרים טבעיים מ 1 עד k, ולכן נוכל ליצור מערך נפרד, בשם C, בגודל k איברים (ונאתחל אותו כך שיכיל את הספרה 0 בכל תא). לאחר מכן נעבור על מערך הקלט ובכל פעם שנקרא איבר ממערך הקלט נעלה באחד את האינדקס שלו במערך C (המערך שיצרנו) (עד כאן ביצענו את שלב a באיור למעלה).

בלולאה השלישית נעבור על המערך C, ונכניס במקום כל אינדקס את הסכום של עצמו פלוס הערך שבאינדקס שלפניו (כך נקבל כמה איברים יש לפניו ונכלול בחישוב את מספר האיברים הקיימים של אותו ערך) (שלב b).

בלולאה הרביעית נתחיל להכניס את כל האיברים למערך חדש B (בגודל של מערך הקלט - A), על ידי כך שנעבור שוב על מערך A (הפעם מהסוף להתחלה) ובכל פעם שנראה ערך, נלך לאותו אינדקס של הערך במערך C (ז"א שאם ראינו את הערך 3 במערך A נלך לאינדקס 3 במערך C) ונכניס את מס' האינקדס (3) למערך B לפי הערך שבמערך C (אם באינדקס 3 במערך C רשום את הערך 7, נכניס את המספר 3 לתא (אינדקס) מספר 7 במערך B).

וכך הלאה עד שנסיים לעבור על מערך A, ונקבל את מערך B שהינו העתק של מערך A ,אך ממוין.

> _שימו לב:_ תכונה חשוב של מיון זה היא שהוא יציב, ז"א שהוא לא משנה את הסדר היחסי בין איברים זהים במיון (אם ישנו 3 פעמים את הערך 5, ולצרוך הדוגמה נקרא להם 5.1, 5.2, 5.3 (לפי הסדר שלהם בקלט), אנו נקבל את אותו הסדר 5.1, 5.2, 5.3 גם במערך הפלט).

### עקרון האלגוריתם:
* ניצור מערך בגודל k ונאתחל אותו ל0 (שורות 1-2).
* נעבור על מערך הקלט וכל פעם שנקרא ממנו איבר, נעלה באחד את אותו האינדקס במערך שיצרנו (שורות 3-4).
* נעבור על המערך שיצרנו ולכל אינדקס נכניס את הסכום של עצמו ועוד הערך שלפניו (שורות 5-6).
* נכניס את האיברים למערך החדש (ע"י כך שנעבור על המערך המקורי מהסוף להתחלה ובכל פעם שנראה ערך נלך לאותו האינדקס של הערך במערך שיצרנו ונכניס את מספר האינדקס למערך החדש במיקום של הערך שבמערך שיצרנו) (שורות 7-9).

&nbsp;

### הקוד של האלגוריתם ב [פסאודו קוד](http://en.wikipedia.org/wiki/Pseudocode)

```c
COUNTING-SORT (A, B, k)
    for (i <- 0 to k) do
        C[i] <- 0
    for (j <- 1 to length[A]) do
        C[A[j]] <- C[A[j]] + 1     // C[i] now contains the number of elements equal to i.
    for (i <- 1 to k) do
        C[i] <- C[i] + C[i - 1]     // C[i] now contains the number of elements less then or equal to i.
    for (j <- length[A] downto 1) do
        B[C[A[j]]] <- A[j]
        C[A[j]] <- C[A[j]] - 1
```

&nbsp;

### הסבר מופשט:

לאלגוריתם יש 4 חלקים, יצירת מערך עזר בגודל k ואתחולו. מעבר על מערך הקלט, קריאה של האיברים ממנו והגדלה של האינדקסים במערך שיצרנו בחלק הקודם. מעבר על המערך שיצרנו וחישוב המיקום של כל איבר. ולבסוף הכנסת האיברים למערך חדש (באותו הגודל כמו מערך הקלט), אותו אנו מוצאים כפלט.

&nbsp;


### הרצה של הקוד לשם המחשה של השלבים המתבצעים:

1. נקבל מערך: {3, 3, 5, 3, 0, 2, 0 }.
2. שורות 1: נעבור על המערך C מ0 ועד k (הספרה הגדולה ביותר במערך הקלט A, שהיא גם הספרה שקובעת מה יהיה גודל המערך C):
  * שורה 2: נאתחל את לתא הנוכחי לספרה 0.
3. כרגע מערך הקלט A יראה אותו הדבר, מערך הפלט B עדיין יהיה ריק, ומערך העזר C יהיה { 0, 0, 0, 0, 0, 0 }.
4. שורה 3: נעבור על מערך הקלט A:
  * שורה 4: נגדיל ב1 את הערך בתא מספר [A[j במערך C (ז"א אם בתא שאנו מסתכלים עליו כעת קיים הערך 2, נלך לתא מס' 2 במערך C ונגדיל אותו ב1).
5. מערך הקלט A כמובן לא ישתנה, מערך הפלט B עדיין יהיה ריק, ומערך העזר C  יהיה {1, 0, 3, 1, 0, 2 }.
6. שורה 5: נעבור על מערך C:
  * שורה 6: נכניס לתא את הערך של עצמו פלוס הערך הקיים בתא שלפנינו (מה שיתן לנו את המיקום בו צריך להיות מס' האינדקס במערך המקורי).
7. מערך הקלט A כמובן לא ישתנה, מערך הפלט B עדיין יהיה ריק, ומערך העזר C  יהיה {7, 6, 6, 3, 2, 2 }.
8. שורה 7: נעבור על מערך הקלט A מהסוף להתחלה:
  * שורה 8: נכניס את הערך ב [A[j למערך B בתא מס' [[C[A[j, ז"א נקרא את הערך ב[A[j ולדוגמה התא האחרון במערך שלנו מכיל את הערך 3 (ונתייחס אליו כרגע) אז נלך לתא מס' 3 במערך C ונקרא את הערך הנמצא שם (שבמקרה זה 6) ולכן נשים את 3 בתא מס' 6 במערך B).
  * שורה 9: וכעת נעדכן את הערך בC במיקום [A[j להיות פחות 1, ז"א כדי שלא נדרוס ערכים. אם נמשיך בדוגמה בשורה מעל, אז לאחר שהצבנו בB, יש לעדכן בC את הערך בתא מס' 3, להיות מ6 ל5 .
  * לאחר ביצוע של 2 השורות האלו מערך הקלט A כמובן לא ישתנה, מערך הפלט B יהיה {-, 3, -, -, -, -, - }, ומערך העזר C יהיה {7, 6, 5, 3, 2, 2 }.
9. לאחר ביצוע כל הלולאה, מערך הקלט A לא ישתנה, מערך הפלט B יהיה {5, 3, 3, 3, 2, 0, 0 }, ומערך העזר C יהיה {6, 6, 3, 2, 2, 0 }.

**סיבוכיות זמן ריצה: (O(n + k.**