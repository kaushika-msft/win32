---
title: About Text Object Model
description: This topic provides a high-level overview of the TOM.
ms.assetid: 'e304ec18-ec2e-4ea7-91c6-6f6ab63b72ae'
---

# About Text Object Model

The Text Object Model (TOM) defines a set of text manipulation interfaces that are supported in varying degrees by several Microsoft text solutions, including the rich edit control. This topic provides a high-level overview of the TOM. It discusses the following topics.

-   [TOM Version 2 Objects](#tom-version-2-objects)
-   [TOM Interface Conventions](#tom-interface-conventions)
-   [The tomBool Type](#the-tombool-type)
-   [Math BuildUp and Build Down](#math-buildup-and-build-down)
-   [TOM RTF](#tom-rtf)
-   [Finding Rich Text](#finding-rich-text)
-   [TOM Accessibility](#tom-accessibility)
    -   [Interface from Running Object Table](#interface-from-running-object-table)
    -   [Interface from Window Messages](#interface-from-window-messages)
    -   [Accessibility Oriented Methods](#accessibility-oriented-methods)
-   [Character Match Sets](#character-match-sets)

## TOM Version 2 Objects

TOM version 2 (TOM 2) extends the original text object model; the new interfaces are derived from the old ones. The updated TOM API includes support for new character and paragraph format properties, a table model, multiple selection, and inline object support for math and ruby.

The top-level TOM 2 object is defined by the [**ITextDocument2**](itextdocument2.md) interface, which has methods for creating and retrieving objects lower in the object hierarchy. For simple plain-text processing, you can obtain an [**ITextRange2**](itextrange2.md) object from an **ITextDocument2** object and do most everything with that. If you need to add rich-text formatting, you can obtain [**ITextFont2**](itextfont2.md) and [**ITextPara2**](itextpara2.md) objects from an **ITextRange2** object. **ITextFont2** provides the programming equivalent of the Microsoft Word format-font dialog, and **ITextPara2** provides the equivalent of the Word format-paragraph dialog.

In addition to these three lower-level objects, TOM 2 has a selection object ([**ITextSelection2**](itextselection2.md)), which is an [**ITextRange2**](itextrange2.md) object with selection highlighting and some UI-oriented methods.

The range and selection objects include screen-oriented methods that enable programs to examine text on screen or text that could be scrolled onto the screen. These capabilities help make text accessible to people with impaired vision, for example.

Each interface that has the 2 suffix inherits from the corresponding interface without the 2 suffix. For example, [**ITextDocument2**](itextdocument2.md) inherits from [**ITextDocument**](itextdocument.md).

The TOM 2 objects have the following hierarchy.

``` syntax
ITextDocument2         Top-level editing object
    ITextRange2        Primary text interface: a range of text
        ITextFont2     Character-attribute interface
        ITextPara2     Paragraph-attribute interface
        ITextRow       Table interface
    ITextSelection2    Screen highlighted text range
        ITextRange2    Selection inherits all range methods
    ITextDisplays      Displays collection (not yet defined)
    ITextStrings       Rich-text strings collection
    ITextStoryRanges2  Enumerator for stories in document
```

An [**ITextDocument2**](itextdocument2.md) object describes one or more contiguous ranges of text called *stories*. Stories represent various parts of a document, such as the main text of the document, headers and footers, footnotes, annotations, and rich-text scratch pads. A scratch pad story is used when translating between linearly formatted math expressions and a built-up form. A scratch pad story is also used when saving the contents of a range that is the current copy source when the contents are about to be changed.

An [**ITextRange2**](itextrange2.md) object is defined by its start and end character-position offsets and a story object. It does not exist independently of its parent story object, although its text can be copied to the clipboard or to other targets. A text range object is different from spreadsheet and other range objects, which are defined by other kinds of offsets; for example, row/column or graphics position (x, y). A text range object can modify itself in various ways, can return a duplicate of itself, and it can be commanded to copy its start and end character positions and its story pointer to the current selection.

An explicit story object is not needed, since an [**ITextRange**](itextrange.md) object can always be created to represent any given story. In particular, the [**ITextDocument**](itextdocument.md) object can create an [**ITextStoryRanges**](itextstoryranges.md) object to enumerate the stories in the document in terms of ranges with start and end character position values that describe complete stories (such as, 0 and **tomForward**).

With an [**ITextStoryRanges2**](itextstoryranges2.md) object, an explicit story object is not needed, since the each story is described by an [**ITextRange2**](itextrange2.md) object. In particular, the [**ITextDocument2**](https://msdn.microsoft.com/library/windows/desktop/hh768436) object can create an [**ITextStoryRanges2**](https://msdn.microsoft.com/library/windows/desktop/hh768722) object to enumerate the stories in the document in terms of ranges with start and end character position values that describe complete stories (such as, 0 and **tomForward**).

The [**ITextRow**](itextrow.md) interface together with the [**ITextRange::Move**](itextrange-move.md) and [**ITextRange::Expand**](itextrange-expand.md) methods give the capability to insert, query, and change tables.

## TOM Interface Conventions

All TOM methods return **HRESULT** values. In general, the TOM methods return the following standard values.

-   E\_OUTOFMEMORY
-   E\_INVALIDARG
-   E\_NOTIMPL
-   E\_FILENOTFOUND
-   E\_ACCESSDENIED
-   E\_FAIL
-   CO\_E\_RELEASED
-   NOERROR (same as S\_OK)
-   S\_FALSE

Be aware that if the editing instance associated with a TOM object such as [**ITextRange**](itextrange.md) is deleted, the TOM object becomes useless, and all its methods return CO\_E\_RELEASED.

In addition to the **HRESULT** return values, many methods include out parameters, which are pointers used to return values. For all interfaces, you should check all pointer parameters to ensure that they are nonzero before using them. If you pass a null value to a method that requires a valid pointer, the method returns E\_INVALIDARG. Optional out pointers with null values are ignored.

Use methods with Get and Set prefixes to get and set properties. Boolean variables use **tomFalse** (0) for **FALSE**, and **tomTrue** (-1) for **TRUE**.

TOM constants are defined in the [**tomConstants**](tomconstants.md) enumeration type and begin with the prefix *tom*, for example **tomWord**.

## The tomBool Type

Many TOM methods use a special type of variable called "tomBool" for rich-text attributes that have binary states. The tomBool type is different from the **Boolean** type because it can take four values: **tomTrue**, **tomFalse**, **tomToggle**, and **tomUndefined**. The **tomTrue** and **tomFalse** values indicate true and false. The **tomToggle** value is used to toggle a property. The **tomUndefined** value, more traditionally called NINCH, is a special no-input, no-change value that works with longs, floats, and **COLORREF**s. For strings, **tomUndefined** (or NINCH) is represented by the null string. For property setting operations, using **tomUndefined** does not change the target property. For property getting operations, **tomUndefined** means that the characters in the range have different values (it gives the grayed check box in property dialog boxes).

## Math BuildUp and Build Down

You can use the [**ITextRange2::BuildUpMath**](itextrange2-buildupmath.md) method to convert linearly formatted math expressions into built-up versions. The [**ITextRange2::Linearize**](itextrange2-linearize.md) method does the opposite conversion, called linearization or build down, to convert built-up versions of math expressions back to linear format. The math build down capability is useful when you need to export plain text or to enable certain types of editing.

## TOM RTF

In TOM, rich-text exchange can be accomplished by sets of explicit method calls or by transfers of rich text in the Rich Text Format (RTF). This section gives tables of RTF control words for paragraph properties and for character properties.

**TOM RTF Paragraph Control Words**

| Control word   | Meaning                                                                                                                                                                                                                                                                        |
|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| \\ fi *n*      | First-line indent (the default is zero).                                                                                                                                                                                                                                       |
| \\ keep        | Keep paragraph intact.                                                                                                                                                                                                                                                         |
| \\ keepn       | Keep with the next paragraph.                                                                                                                                                                                                                                                  |
| \\ li *n*      | Left indent (the default is zero).                                                                                                                                                                                                                                             |
| \\ noline      | No line numbering.                                                                                                                                                                                                                                                             |
| \\ nowidctlpar | Turn off widow/orphan control.                                                                                                                                                                                                                                                 |
| \\ pagebb      | Break page before paragraph.                                                                                                                                                                                                                                                   |
| \\ par         | New paragraph.                                                                                                                                                                                                                                                                 |
| \\ pard        | Resets to default paragraph properties.                                                                                                                                                                                                                                        |
| \\ ql          | Left aligned (the default).                                                                                                                                                                                                                                                    |
| \\ qr          | Right aligned.                                                                                                                                                                                                                                                                 |
| \\ qj          | Justified.                                                                                                                                                                                                                                                                     |
| \\ qc          | Centered.                                                                                                                                                                                                                                                                      |
| \\ ri *n*      | Right indent (the default is zero).                                                                                                                                                                                                                                            |
| \\ s *n*       | Style *n*.                                                                                                                                                                                                                                                                     |
| \\ sa *n*      | Space after (the default is zero).                                                                                                                                                                                                                                             |
| \\ sb *n*      | Space before (the default is zero).                                                                                                                                                                                                                                            |
| \\ sl *n*      | If missing or if *n*=1000, line spacing is determined by the tallest character in the line (single-line spacing); if *n*&gt; zero, at least this size is used; if *n* is &lt; zero, exactly \|*n*\| is used. The line spacing is multiple-line spacing if \\ slmult 1 follows. |
| \\ slmult *m*  | Follows \\ sl. *m* = zero: At Least or Exactly line spacing as described by \\ sl *n*. *m* = 1: line spacing = *n*/240 times single-line spacing.                                                                                                                              |
| \\ tb *n*      | Bar tab position, in twips, from the left margin.                                                                                                                                                                                                                              |
| \\ tldot       | Tab leader dots.                                                                                                                                                                                                                                                               |
| \\ tleq        | Tab leader equal sign.                                                                                                                                                                                                                                                         |
| \\ tlhyph      | Tab leader hyphens.                                                                                                                                                                                                                                                            |
| \\ tlth        | Tab leader thick line.                                                                                                                                                                                                                                                         |
| \\ tlul        | Tab leader underline.                                                                                                                                                                                                                                                          |
| \\ tqc         | Centered tab.                                                                                                                                                                                                                                                                  |
| \\ tqdec       | Decimal tab.                                                                                                                                                                                                                                                                   |
| \\ tqr         | Flush-right tab.                                                                                                                                                                                                                                                               |
| \\ tx *n*      | Tab position, in twips, from the left margin.                                                                                                                                                                                                                                  |



 

**TOM RTF Character Format Control Words**

| Control word     | Meaning                                                                                                                                                                                                                                  |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| \\ animation *n* | Sets animation type to *n*.                                                                                                                                                                                                              |
| \\ b             | Bold.                                                                                                                                                                                                                                    |
| \\ caps          | All capitals.                                                                                                                                                                                                                            |
| \\ cf *n*        | Foreground color (the default is **tomAutocolor**).                                                                                                                                                                                      |
| \\ cs *n*        | Character style *n*.                                                                                                                                                                                                                     |
| \\ dn *n*        | Subscript position in half-points (the default is 6).                                                                                                                                                                                    |
| \\ embo          | Embossed.                                                                                                                                                                                                                                |
| \\ f *n*         | Font number, *n* refers to an entry in the font table.                                                                                                                                                                                   |
| \\ fs *n*        | Font size in half-points (the default is 24).                                                                                                                                                                                            |
| \\ highlight *n* | Background color (the default is **tomAutocolor**).                                                                                                                                                                                      |
| \\ i             | Italic.                                                                                                                                                                                                                                  |
| \\ impr          | Imprint.                                                                                                                                                                                                                                 |
| \\ lang *n*      | Applies a language to a character. *n* is a number corresponding to a language. The \\ plain control word resets the language property to the language defined by \\ deflang *n* in the document properties.                             |
| \\ nosupersub    | Turns off superscript or subscript.                                                                                                                                                                                                      |
| \\ outl          | Outline.                                                                                                                                                                                                                                 |
| \\ plain         | Resets character formatting properties to a default value defined by the application. The associated character-formatting properties (described in the section Associated Character Properties in the RTF specification) are also reset. |
| \\ scaps         | Small capitals.                                                                                                                                                                                                                          |
| \\ shad          | Shadow.                                                                                                                                                                                                                                  |
| \\ strike        | Strikethrough.                                                                                                                                                                                                                           |
| \\ sub           | Applies subscript to text and reduces point size according to font information.                                                                                                                                                          |
| \\ super         | Applies superscript to text and reduces point size according to font information.                                                                                                                                                        |
| \\ ul            | Continuous underline. \\ ul0 turns off all underlining.                                                                                                                                                                                  |
| \\ uld           | Dotted underline.                                                                                                                                                                                                                        |
| \\ uldb          | Double underline.                                                                                                                                                                                                                        |
| \\ ulnone        | Stops all underlining.                                                                                                                                                                                                                   |
| \\ ulw           | Word underline.                                                                                                                                                                                                                          |
| \\ up *n*        | Superscript position in half-points (the default is 6).                                                                                                                                                                                  |
| \\ v             | Hidden text.                                                                                                                                                                                                                             |



 

## Finding Rich Text

You can use TOM methods to find rich text as defined by a range of text. Finding such rich text exactly is often needed in word processing, although it has never been fulfilled in a "what you see is what you get" (WYSIWYG) word processor. There is clearly a larger domain of rich-text matching that allows for some character formatting properties to be ignored (or to include paragraph formatting and/or object content), but such generalizations are beyond the scope of this section.

One purpose for this functionality is to use a rich-text **Find** dialog box to define the rich text you want to locate in a document. The dialog box would be implemented using a rich edit control and TOM methods would be used to carry out the search through the document. You could either copy the desired rich text from the document into the **Find** dialog box, or enter and format it directly in the **Find** dialog box.

The following example shows how to use TOM methods to find text containing combinations of exact character formatting. The algorithm searches for the plain text in the match range, which is named `pr1`. If the plain text is found, it is pointed to by a trial range, which is named `pr2`. Then, two insertion-point ranges (`prip1` and `prip2`) are used to walk through the trial range comparing its character formatting to that of `pr1`. If they match exactly, the input range (given by `ppr`) is updated to point at the trial range's text and the function returns the count of characters in the matched range. Two [**ITextFont**](itextfont.md) objects, `pf1` and `pf2`, are used in the character-formatting comparison. They are attached to the insertion-point ranges `prip1` and `prip2`.


```
LONG FindRichText (
    ITextRange **ppr,             // Ptr to range to search
    ITextRange *pr1)              // Range with rich text to find
{
    BSTR        bstr;             // pr1 plain-text to search for
    LONG        cch;              // Text string count
    LONG        cch1, cch2;       // tomCharFormat run char counts
    LONG        cchMatch = 0;     // Nothing matched yet
    LONG        cp;               // Handy char position
    LONG        cpFirst1;         // pr1 cpFirst
    LONG        cpFirst2;         // pr2 cpFirst
    ITextFont  *    pf1, *pf      // Fonts corresponding to IPs prip1 and prip2
    ITextRange *pr2;              // Range duplicate to search with
    ITextRange *prip1, *prip      // Insertion points to walk pr1, pr2

    if (!ppr || !*ppr || !pr1)
        return E_INVALIDARG;

    // Initialize range and font objects used in search
    if ((*ppr)->GetDuplicate(&amp;pr2)    != NOERROR ||
        pr1->GetDuplicate(&amp;prip1)     != NOERROR ||
        pr2->GetDuplicate(&amp;prip2)     != NOERROR ||
        prip1->GetFont(&amp;pf1)          != NOERROR ||
        prip2->GetFont(&amp;pf2)          != NOERROR ||
        pr1->GetText(&amp;bstr)           != NOERROR )
    {
        return E_OUTOFMEMORY;
    }

    pr1->GetStart(&amp;cpFirst1);

    // Keep searching till rich text is matched or no more plain-text hits
    while(!cchMatch &amp;&amp; pr2->FindText(bstr, tomForward, 0, &amp;cch) == NOERROR)
    {
        pr2->GetStart(&amp;cpFirst2);                 // pr2 is a new trial range
        prip1->SetRange(cpFirst1, cpFirst1);      // Set up IPs to scan match
        prip2->SetRange(cpFirst2, cpFirst2);      //  and trial ranges

        while(cch > 0 &amp;&amp;
            pf1->IsEqual(pf2, NULL) == NOERROR)   // Walk match & trial ranges
        {                                         //  together comparing font
            prip1->GetStart(&amp;cch1);               //  properties
            prip1->Move(tomCharFormat, 1, NULL);
            prip1->GetStart(&amp;cp);
            cch1 = cp - cch1;                     // cch of next match font run

            prip2->GetStart(&amp;cch2);
            prip2->Move(tomCharFormat, 1, NULL);
            prip2->GetStart(&amp;cp);
            cch2 = cp - cch2;                      // cch of next trial font run

            if(cch1 < cch)                         // There is more to compare
            {
                if(cch1 != cch2)                   // Different run lengths:
                    break;                         //  no formatting match
                cch = cch - cch1;                  // Matched format run
            }
            else if(cch2 < cch)                    // Trial range format run too
                break;                             //  short

            else                                   // Both match and trial runs
            {                                      //  reach at least to match
                pr2->GetEnd(&amp;cp);                  //  text end: rich-text match
                (*ppr)->SetRange(cpFirst2, cp)     // Set input range to hit
                cchMatch = cp - cpFirst2;          //  coordinates and return
                break;                             //  length of matched string
            }
        }
    }
    pr2->Release();
    prip1->Release();
    prip2->Release();
    pf1->Release();
    pf2->Release();
    SysFreeString(bstr);

    return cchMatch;
}
```



## TOM Accessibility

TOM provides accessibility support through the [**ITextSelection**](itextselection.md) and [**ITextRange**](itextrange.md) interfaces. This section describes methods that are useful for accessibility as well as how a program can determine the *x*, *y* screen position of an object.

Since UI-based accessibility programs typically work with the screen and the mouse, a common concern is to find the corresponding [**ITextDocument**](itextdocument.md) interface for the current mouse location (in screen coordinates). The following sections present two ways to determine the proper interface:

-   [Through the running-object table](#interface-from-running-object-table)
-   Through the [**EM\_GETOLEINTERFACE**](em-getoleinterface.md) message, which works for windowed rich edit instances, provided the client is in the same process space (no *marshaling* is needed)

For more information, see the Microsoft Active Accessibility specification. After you obtain an object from a screen position, you can use for an [**ITextDocument**](itextdocument.md) interface and call the [**RangeFromPoint**](itextdocument-rangefrompoint.md) method to get an empty range object at the cp corresponding to the screen position.

### Interface from Running Object Table

A running object table (ROT) tells what object instances are active. By querying this table, you can accelerate the process of connecting a client to an object when the object is already running. Before programs can access TOM interfaces through the running object table, a TOM instance with a window needs to register in the ROT using a moniker. You construct the moniker from a string containing the hexadecimal value of its [**HWND**](https://msdn.microsoft.com/library/windows/desktop/aa383751#hwnd). The following code sample shows how to do this.


```
// This TOM implementation code is executed when a new windowed 
// instance starts up. 
// Variables with leading underscores are members of this class.

HRESULT hr;
OLECHAR szBuf[10];            // Place to put moniker
MONIKER *pmk;

hr = StringCchPrintf(szBuff, 10, "%x", _hwnd);
if (FAILED(hr))
{
    //
    // TODO: write error handler
    //
}
CreateFileMoniker(szBuf, &amp;pmk);
OleStdRegisterAsRunning(this, pmk, &amp;_dwROTcookie);
....................
 
// Accessibility Client: 
//    Find hwnd for window pointed to by mouse cursor.

GetCursorPos(&amp;pt);
hwnd = WindowFromPoint(pt);

// Look in ROT (running object table) for an object attached to hwnd

hr = StringCchPrintf(szBuff, 10, "%x", hwnd);
if (FAILED(hr))
{
    //
    // TODO: write error handler
    //
}
CreateFileMoniker(szBuf, &amp;pmk);
CreateBindContext(0, &amp;pbc);
pmk->BindToObject(pbc, NULL, IID_ITextDocument, &amp;pDoc);
pbc->Release();

if( pDoc )
{
    pDoc->RangeFromPoint(pt.x, pt.y, &amp;pRange);
    // ...now do whatever with the range pRange
}
```



### Interface from Window Messages

The [**EM\_GETOLEINTERFACE**](em-getoleinterface.md) message provides another way to obtain an [**IUnknown**](https://msdn.microsoft.com/library/windows/desktop/ms680509) interface for an object at a given screen position. As described in [Interface from Running Object Table](#interface-from-running-object-table), you get an [**HWND**](https://msdn.microsoft.com/library/windows/desktop/aa383751#hwnd) for the screen position and then send this message to that **HWND**. The **EM\_GETOLEINTERFACE** message is rich edit-specific and returns a pointer to an [**IRichEditOle**](iricheditole.md) interface in the variable addressed by *lParam*.

**Tip** If a pointer is returned (be sure to set the object to which *lParam* points to null before sending the message), you can call its [**IUnknown::QueryInterface**](https://msdn.microsoft.com/library/windows/desktop/ms682521) method to obtain an [**ITextDocument**](itextdocument.md) interface. The following code sample illustrates this approach.


```
    HWND    hwnd;
    ITextDocument *pDoc;
    ITextRange *pRange;
    POINT    pt;
    IUnknown *pUnk = NULL;
    
    GetCursorPos(&amp;pt);
    hwnd = WindowFromPoint(pt);
    SendMessage(hwnd, EM_GETOLEINTERFACE, 0, (LPARAM)&amp;pUnk);
    if(pUnk &amp;&amp; 
        pUnk->QueryInterface(IID_ITextDocument, &amp;pDoc) == NOERROR)
    {
        pDoc->RangeFromPoint(pt.x, pt.y, &amp;pRange);
        //  ... continue with rest of program
    }
```



### Accessibility Oriented Methods

Some TOM methods are particularly useful for navigating around the screen, while other TOM methods enhance what you can do when you arrive at places of interest. The following table describes the most useful methods.



| Method                                                 | How it promotes accessibility                                                                                                                                                                 |
|--------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [**GetSelection**](itextdocument-getselection.md)     | This method gets the active selection that can be used for a variety of view-oriented purposes, such as highlighting text and scrolling.                                                      |
| [**RangeFromPoint**](itextdocument-rangefrompoint.md) | When used on an active selection, this method is guaranteed to get a range associated with a particular view.                                                                                 |
| [**Expand**](itextrange-expand.md)                    | Enlarges a text range so that any partial units it contains are completely contained. For example, `Expand(tomWindow)` expands the range to include the visible portion of the range's story. |
| [**GetDuplicate**](itextrange-getduplicate.md)        | When used on an active selection, this method is guaranteed to get a range associated with a particular view. See the description of [**RangeFromPoint**](itextdocument-rangefrompoint.md).  |
| [**GetPoint**](itextrange-getpoint.md)                | Gets the screen coordinates for the start or end character position in the text range.                                                                                                        |
| [**ScrollIntoView**](itextrange-scrollintoview.md)    | Scrolls a text range into view.                                                                                                                                                               |
| [**SetPoint**](itextrange-setpoint.md)                | Selects text at or up through a specified point.                                                                                                                                              |



 

## Character Match Sets

The *variant* parameter of the various **Move**\* methods in [**ITextRange**](itextrange.md), such as [**MoveWhile**](itextrange-movewhile.md) and [**MoveUntil**](itextrange-moveuntil.md), can take an explicit string or a character-match set 32-bit index. The indexes are defined by either Unicode ranges or [**GetStringTypeEx**](https://msdn.microsoft.com/library/windows/desktop/dd318118) character sets. The Unicode range starting at *n* and of length *l* (&lt; 32768) is given by the index *n* + (*l &lt;*&lt; 16) + 0x80000000. For example, basic Greek letters are defined by CR\_Greek = 0x805f0370 and printable ASCII characters are defined by CR\_ASCIIPrint = 0x805e0020. In addition, the **MoveWhile** and **MoveUntil** methods let you rapidly bypass a span of characters in any **GetStringTypeEx** character set, or in a span of characters that is not in any one of these character sets.

The [**GetStringTypeEx**](https://msdn.microsoft.com/library/windows/desktop/dd318118) sets are specified by the values for *Ctype1*, *Ctype2*, and *Ctype3*, and are defined as follows.



| Cset                 | Meaning                          |
|----------------------|----------------------------------|
| *Ctype1*             | Combination of CT\_CTYPE1 types. |
| *Ctype2* + tomCType2 | Any CT\_CTYPE2 type.             |
| *Ctype3* + tomCType3 | Combination of CT\_CTYPE3 types. |



 

Specifically, *Ctype1* can be any combination of the following.



| Ctype1 name | Value  | Meaning                                                           |
|-------------|--------|-------------------------------------------------------------------|
| C1\_UPPER   | 0x0001 | Uppercase.                                                        |
| C1\_LOWER   | 0x0002 | Lowercase.                                                        |
| C1\_DIGIT   | 0x0004 | Decimal digits.                                                   |
| C1\_SPACE   | 0x0008 | Space characters.                                                 |
| C1\_PUNCT   | 0x0010 | Punctuation.                                                      |
| C1\_CNTRL   | 0x0020 | Control characters.                                               |
| C1\_BLANK   | 0x0040 | Blank characters.                                                 |
| C1\_XDIGIT  | 0x0080 | Hexadecimal digits.                                               |
| C1\_ALPHA   | 0x0100 | Any linguistic character (alphabetic, syllabary, or ideographic). |
| C1\_DEFINED | 0x0200 | A defined character, but not one of the other C1\_\* types.       |



 

The *Ctype2* types support proper layout of Unicode text. The direction attributes are assigned so that the bidirectional layout algorithm standardized by Unicode produces accurate results. These types are mutually exclusive. For more information about the use of these attributes, see *The Unicode Standard: Worldwide Character Encoding, Volumes 1 and 2*, Addison-Wesley Publishing Company: 1991, 1992.



| CType2 name          | Value | Meaning                          |
|----------------------|-------|----------------------------------|
| Strong:              |       |                                  |
| C2\_LEFTTORIGHT      | 0x1   | Left to right.                   |
| C2\_RIGHTTOLEFT      | 0x2   | Right to left.                   |
| Weak:                |       |                                  |
| C2\_EUROPENUMBER     | 0x3   | European number, European digit. |
| C2\_EUROPESEPARATOR  | 0x4   | European numeric separator.      |
| C2\_EUROPETERMINATOR | 0x5   | European numeric terminator.     |
| C2\_ARABICNUMBER     | 0x6   | Arabic number.                   |
| C2\_COMMONSEPARATOR  | 0x7   | Common numeric separator.        |
| Neutral:             |       |                                  |
| C2\_BLOCKSEPARATOR   | 0x8   | Block separator.                 |
| C2\_SEGMENTSEPARATOR | 0x9   | Segment separator.               |
| C2\_WHITESPACE       | 0xA   | White space.                     |
| C2\_OTHERNEUTRAL     | 0xB   | Other neutrals.                  |
| Not applicable:      |       |                                  |
| C2\_NOTAPPLICABLE    | 0x0   | No implicit direction.           |



 

The *Ctype3* types are intended to be placeholders for extensions to the POSIX types required for general text processing or for the standard C library functions.



| CType3 name       | Value  | Meaning                                                             |
|-------------------|--------|---------------------------------------------------------------------|
| C3\_NONSPACING    | 0x1    | Nonspacing mark.                                                    |
| C3\_DIACRITIC     | 0x2    | Diacritic nonspacing mark.                                          |
| C3\_VOWELMARK     | 0x4    | Vowel nonspacing mark.                                              |
| C3\_SYMBOL        | 0x8    | Symbol.                                                             |
| C3\_KATAKANA      | 0x10   | Katakana character.                                                 |
| C3\_HIRAGANA      | 0x20   | Hiragana character.                                                 |
| C3\_HALFWIDTH     | 0x40   | Half-width character.                                               |
| C3\_FULLWIDTH     | 0x80   | Full-width character.                                               |
| C3\_IDEOGRAPH     | 0x100  | Ideographic character.                                              |
| C3\_KASHIDA       | 0x200  | Arabic Kashida character.                                           |
| C3\_ALPHA         | 0x8000 | All linguistic characters (alphabetic, syllabary, and ideographic). |
| C3\_NOTAPPLICABLE | 0x0    | Not applicable.                                                     |



 

An Edit Development Kit (EDK) could include *pVar* index definitions for the following ranges described in the Unicode Standard.



| Character set         | Unicode Range | Character set             | Unicode Range |
|-----------------------|---------------|---------------------------|---------------|
| ASCII                 | 0x0—0x7f      | ANSI                      | 0x0—0xff      |
| ASCIIPrint            | 0x20—0x7e     | Latin1                    | 0x20—0xff     |
| Latin1Supp            | 0xa0—0xff     | LatinXA                   | 0x100—0x17f   |
| LatinXB               | 0x180—0x24f   | IPAX                      | 0x250—0x2af   |
| SpaceMod              | 0x2b0—0x2ff   | Combining                 | 0x300—0x36f   |
| Greek                 | 0x370—0x3ff   | BasicGreek                | 0x370—0x3cf   |
| GreekSymbols          | 0x3d0—0x3ff   | Cyrillic                  | 0x400—0x4ff   |
| Armenian              | 0x530—0x58f   | Hebrew                    | 0x590—0x5ff   |
| BasicHebrew           | 0x5d0—0x5ea   | HebrewXA                  | 0x590—0x5cf   |
| HebrewXB              | 0x5eb—0x5ff   | Arabic                    | 0x600—0x6ff   |
| BasicArabic           | 0x600—0x652   | ArabicX                   | 0x653—0x6ff   |
| Devangari             | 0x900—0x97f   | Bangla (formerly Bengali) | 0x980—0x9ff   |
| Gurmukhi              | 0xa00—0xa7f   | Gujarati                  | 0xa80—0xaff   |
| Odia (formerly Oriya) | 0xb00—0xb7f   | Tamil                     | 0xb80—0xbff   |
| Teluga                | 0xc00—0xc7f   | Kannada                   | 0xc80—0xcff   |
| Malayalam             | 0xd00—0xd7f   | Thai                      | 0xe00—0xe7f   |
| Lao                   | 0xe80—0xeff   | GeorgianX                 | 0x10a0—0xa0cf |
| BascGeorgian          | 0x10d0—0x10ff | Jamo                      | 0x1100—0x11ff |
| LatinXAdd             | 0x1e00—0x1eff | GreekX                    | 0x1f00—0x1fff |
| GenPunct              | 0x2000—0x206f | Superscript               | 0x2070—0x207f |
| Subscript             | 0x2080—0x208f | SuperSubscript            | 0x2070—0x209f |
| Currency              | 0x20a0—0x20cf | CombMarkSym               | 0x20d0—0x20ff |
| LetterLike            | 0x2100—0x214f | NumberForms               | 0x2150—0x218f |
| Arrows                | 0x2190—0x21ff | MathOps                   | 0x2200—0x22ff |
| MiscTech              | 0x2300—0x23ff | CtrlPictures              | 0x2400—0x243f |
| OptCharRecog          | 0x2440—0x245f | EnclAlphaNum              | 0x2460—x24ff  |
| BoxDrawing            | 0x2500—0x257f | BlockElement              | 0x2580—0x259f |
| GeometShapes          | 0x25a0—0x25ff | MiscSymbols               | 0x2600—0x26ff |
| Dingbats              | 0x2700—0x27bf | CJKSymPunct               | 0x3000—0x303f |
| Hiragana              | 0x3040—0x309f | Katakana                  | 0x30a0—0x30ff |
| Bopomofo              | 0x3100—0x312f | HangulJamo                | 0x3130—0x318f |
| CJLMisc               | 0x3190—0x319f | EnclCJK                   | 0x3200—0x32ff |
| CJKCompatibl          | 0x3300—0x33ff | Han                       | 0x3400—0xabff |
| Hangul                | 0xac00—0xd7ff | UTF16Lead                 | 0xd800—0xdbff |
| UTF16Trail            | 0xdc00—0xdfff | PrivateUse                | 0xe000—0xf800 |
| CJKCompIdeog          | 0xf900—0xfaff | AlphaPres                 | 0xfb00—0xfb4f |
| ArabicPresA           | 0xfb50—0xfdff | CombHalfMark              | 0xfe20—0xfe2f |
| CJKCompForm           | 0xfe30—0xfe4f | SmallFormVar              | 0xfe50—0xfe6f |
| ArabicPresB           | 0xfe70—0xfefe | HalfFullForm              | 0xff00—0xffef |
| Specials              | 0xfff0—0xfffd |                           |               |



 

 

 



