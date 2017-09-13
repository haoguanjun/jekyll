---
layout: post
---

# Chrome compatible with IE

## [window.showModalDialog: What It is and Why You should never use it?](https://www.tjvantoll.com/2012/05/02/showmodaldialog-what-it-is-and-why-you-should-never-use-it/){:target="_blank"}
Ah, 1997. The first browser war was in full force, and Microsoft was busy adding proprietary new features to compete with Netscape Navigator. One of those features was introducing a common OS UI element into the browser - modal dialogs. Internet Explorer 4 launched with a showModalDialog method on the global window object. When called it displays a dialog that the user has to deal with before interacting with the rest of the page.
Fast forward a few years and Internet Explorer had won the war, 95+ percent of us were using IE6. Consequently a whole lot of web applications were designed around many of the proprietary features that IE had added. Interestingly several of these have recently been added to the HTML5 specification including innerHTML, insertAdjacentHTML, outerHTML, and… window.showModalDialog.

So now that window.showModalDialog has been standardized should you be using it?

No.

In general the idea of putting a native dialog implementation into the browser was a really good idea, but window.showModalDialog was a bad implementation that is riddled with issues and poor browser support.

### Modal Dialogs

So why did Microsoft add modal dialogs to begin with? They’re actually a heavily used UI element in most all computer interfaces. Try to shut off your phone, tablet, laptop, etc.. and you’re almost certainly going to be presented with a modal dialog asking you to confirm your decision before being allowed to shut it down. What makes it modal is the fact that you are forced to make a selection before you do anything else.

It’s oftentimes convenient from a usability stand point to get some form of feedback from a user before allowing them to continue. showModalDialog was simply Microsoft’s attempt to bring this UI element to the web.

### Implementation

To use the showModalDialog method you simply call it with a URL.

    window.showModalDialog('http://google.com');
    
This will open up a modal dialog with Google loaded in it. In and of itself this isn’t all that useful. Usually if you’re showing a modal dialog you want to get some information back from it. This is where the window.returnValue comes into play.

#### window.returnValue

    <!-- page.html -->
    <!DOCTYPE html>
    <html>
      <head>
        <script>
          var result = window.showModalDialog('modal.html');
          console.log(result); //'foo'
        </script>
      </head>
      <body>
      </body>
    </html>

    <!-- modal.html -->
    <!DOCTYPE html>
    <html>
      <head>
        <script>
          window.returnValue = 'foo';
          window.close();
        </script>
      </head>
      <body>
      </body>
    </html>
