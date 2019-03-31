# DSFAttributedStringStream

A simple Swift NSAttributedString wrapper to simplify creation of a styled string.

```swift
let attributedString = NSAttributedString.stream {
   $0.set(NSColor.blue)
     .set(NSFont.systemFont(ofSize: 12))
     .append("Blue Text")
}
```

I wanted to be able to use attributed strings within one of my projects, however I kept getting stung with trying to work out offsets (especially regarding complex characters such as emoji, and even moreso when parts of the string were dependent on variable values etc.).  And as the string gets longer, slight changes to the text have major impacts.

I also wanted a _relatively_ safe method for defining the string.  I really like the idea of using HTML style tags, but this would require string parsing to determine offsets and handling parsing errors.

This class is designed as a very simple lightweight NSAttributedString creator class.

```swift
let attributedString = 
   NSAttributedString.stream {
      $0.set(NSFont.boldSystemFont(ofSize: 12))
      $0.append(NSLocalizedString("Important!", comment: "Important text"))
}
```

# API and usage

Just add `DSFAttributedStringStream.swift` to your project

## Adding text and images

### .append(`<text>`)

Appends text to the stream. The text will be styled using the currently active styles. For example,

```swift
let result = NSAttributedString.stream {
   ...
	$0.append("This is a test")
   ...
}
```

### .append(`<image>`)

Appends an image to the stream. This can be either NSImage or UIImage

```swift
let result = NSAttributedString.stream {
   ...
   let myImage = NSImage(named: NSImage.bookmarksTemplateName)!
   $0.append(myImage)
   ...
  }
```

### .endl()

Adds a line break

```swift
stream.append("This is a test").endl().endl()
      .append("Two lines later")
```

### .tab()

Adds a tab character

```swift
stream.append("item1").tab().append("value1").endl()
      .append("item2").tab().append("value2").endl()
```

## Setting styles

Styles are applied from the current position in the string onwards, until the end of the string or they are unset or overridden.

### .set(`<values>`)

Calling `.set()` on the stream allows you to set some characteristics to the current position and future text in the stream.

Calling `.set()` with an attribute already in progress will override the previous `.set()` from the current point onwards

### .unset(`<values>`)

Unsetting a value turns off the style from the current position onwards.

### .unsetAll
Calling `.unsetAll()` turns off all attributes at the current position.

#### Example: Italicize a section of text

```swift
let attributedString = NSAttributedString.stream { stream in
   stream.set(NSColor.textColor)
      .append("This is ")               // Add some plain text
      .set([.obliqueness: 0.1])         // Turn on italic
      .append("italic text ")           // Add some italic text
      .set([.obliqueness: -0.1])        // Override previous .obliqueness call with new value from this point on
      .append("backward italic text")   // Add some backward italic text
      .unset(.obliqueness)              // Turn off backward italic
      .append(". And this is not")      // Add some plain text
```

#### Example: Add a tooltip to some text

```swift
let attributedString = NSAttributedString.stream { 
    $0.set(NSColor.textColor)
    $0.set(NSFont.systemFont(ofSize: NSFont.systemFontSize))
    $0.append("This could be better performed using a ")
    $0.set([.toolTip: "In functional programming, a monad is a design pattern that allows structuring programs generically while automating away boilerplate code needed by the program logic"])
    $0.append("monad")
    $0.set([.baselineOffset: 5])
    $0.set(NSFont.systemFont(ofSize: NSFont.smallSystemFontSize))
    $0.append("1")
    $0.unset([.baselineOffset, .toolTip])
    $0.set(NSFont.systemFont(ofSize: NSFont.systemFontSize))
    $0.append(".")
    $0.endl().endl()
}
```

### .link(`<URL>`, optional `<text>`)

Appends `text` to the stream, applying a link `URL` to the text.  The link attribute is automatically unset at the end of `text`.

```swift
let attributedString = NSAttributedString.stream {
   $0.append("To visit our website, ")
   $0.link(text: "click here", url: URL(string: "https://apple.com")!)
}
```

Or to simply add a link with the link URL

```swift
let attributedString = NSAttributedString.stream {
   $0.link(url: URL(string: "https://swift.org/")!)
}
```

If text is not supplied, the text reflects the URL

## Convenience overloads

### NSShadow

`NSShadow` has a number of attributes which need to be defined upfront before it can be used.

```swift
let attributedString = NSAttributedString.stream { stream in
   stream.set(NSShadow.stream { (shadow) in
                 shadow.shadowColor = NSColor.disabledControlTextColor
                 shadow.shadowOffset = NSSize(width: 1, height: -1)
                 shadow.shadowBlurRadius = 3.0
              })
   stream.append("This is some shadowed title text")
}
```

### NSParagraphStyle


```swift
let attributedString = NSAttributedString.stream { stream in
   stream.set(NSParagraphStyle.stream { style in
				style.alignment = .center
			})
   stream.append("This is some centered text")
}
```

## Styles

You can pre-define styles using a simple class container, and use them to mimic a (basic) style sheet.

For example :-

```swift
class Styles {
   static var H1: [NSAttributedString.Key : Any] {
      return [.font: NSFont.boldSystemFont(ofSize: 24), .foregroundColor: NSColor.red ]
   }
   static var H2: [NSAttributedString.Key : Any] {
      return [.font: NSFont.boldSystemFont(ofSize: 18), .foregroundColor: NSColor.systemPink ]
   }
   static var P: [NSAttributedString.Key : Any] {
      return [.font: NSFont.systemFont(ofSize: NSFont.systemFontSize), .foregroundColor: NSColor.textColor ]
   }
   static var italic: [NSAttributedString.Key : Any] {
      return [ .obliqueness: 0.2 ]
   }
   static func underline(_ style: NSUnderlineStyle) -> [NSAttributedString.Key : Any] {
      return [ .underlineStyle: style.rawValue ]
   }
   static func underline() -> [NSAttributedString.Key : Any] {
      return [ .underlineStyle: NSUnderlineStyle.single.rawValue ]
   }
   static func PBold() -> [NSAttributedString.Key : Any] {
      return [.font: NSFont.boldSystemFont(ofSize: NSFont.systemFontSize), .foregroundColor: NSColor.textColor ]
   }
}
	
let attributedString = NSAttributedString.stream {
   $0.set(Styles.P).set(Styles.underline())
   $0.append("Simple test using basic style sheet")
   $0.unset(Styles.underline()).endl().endl()
   $0.set(Styles.H1).append("Heading").endl()
   $0.set(Styles.H2).append("Subheading").endl()
   $0.set(Styles.P).append("This is the ")
   $0.set(Styles.italic).append("text").unset(Styles.italic)
   $0.append(". ")
}

```

## Objective-C

A simple objective-c example

```objective-c
DSFAttributedStringStream* stream = [[DSFAttributedStringStream alloc] init];

[stream setFont:[NSFont systemFontOfSize:32]];
[stream setColor:[NSColor textColor]];
[stream setStyle:NSUnderlineStyleAttributeName:@(NSUnderlineStyleSingle)];
[stream append:@"Heading"];
[stream unsetStyle:NSUnderlineStyleAttributeName];
[stream append:@" "];
[stream appendScaledImage:[NSImage imageNamed:NSImageNameInfo]:CGSizeMake(28, 28)];
[stream append:@" With "];
[stream setColor:[NSColor systemBlueColor]];
[stream setStyle:NSObliquenessAttributeName:@(0.2)];
[stream append:@"Image"];
[stream unsetAll];

[[stream endl] endl];

NSMutableParagraphStyle* style = [[NSMutableParagraphStyle alloc] init];
[style setAlignment:NSTextAlignmentRight];
[stream setStyle:NSParagraphStyleAttributeName:style];
[stream setColor:[NSColor textColor]];
[stream append:@"﷽"];
[[stream endl] endl];

NSAttributedString* attributedString = [stream attributed];
```

# License

```
MIT License

Copyright (c) 2019 Darren Ford

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

```
