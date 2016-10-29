---
layout: post
title: RandomQuote实现
date: 2016-10-28
categories: blog
tags: [javascript]
description: RandomQuote实现
---


随机语录实现，主要实现了点击时自动变换语录，并且背景颜色与字体颜色变化，而且可以发tweet分享 

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p10.png)

**html代码**            

```
<!DOCTYPE html>
<html>

<head>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css">
    <link rel="stylesheet" type="text/css" href="css/main.css">
    <link rel="icon" href="images/r2d2.ico">
    <title>Build a Random Quote Machine</title>
</head>

<body>
    <div class="quote-box">
        <div class="quote-text">
            <i class="fa fa-quote-left"> </i><span id="text"></span>
        </div>
        <div class="quote-author">
            - <span id="author"></span>
        </div>
        <div class="buttons">
            <a class="button" id="tweet-quote" title="Tweet this quote!" target="_blank">
                <i class="fa fa-twitter"></i>
            </a>
            <button class="button" id="new-quote">New quote</button>
        </div>
    </div>
    <div class="footer"> by <a href="http://whuhan2013.github.io/">Ricardo</a></div>
    <script src='https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js'></script>
    <script src='https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.11.2/jquery-ui.min.js'></script>
    <script src="js/quote.js"></script>
</body>

</html>
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p11.png)

**main.css**         

```
@import url(https://fonts.googleapis.com/css?family=Raleway:400,500);
* {
    margin: 0;
    padding: 0;
    list-style: none;
    vertical-align: baseline;
}

div {
    /*position: relative;*/
    z-index: 2;
}

body {
    background-color: #333;
    color: #333;
    font-family: 'Raleway', sans-serif;
    font-weight: 400;
}

.footer {
    width: 450px;
    text-align: center;
    display: block;
    margin: 15px auto 30px auto;
    font-size: 0.8em;
    color: #fff;
}

.footer a {
    font-weight: 500;
    text-decoration: none;
    color: #fff;
}

.quote-box {
    border-radius: 3px;
    /*position: relative;*/
    margin: 8% auto auto auto;
    width: 450px;
    padding: 40px 50px;
    display: table;
    background-color: #fff;
}

.quote-box .quote-text {
    text-align: center;
    width: 450px;
    height: auto;
    clear: both;
    font-weight: 500;
    font-size: 1.75em;
}

.quote-box .quote-text i {
    font-size: 1.0em;
    margin-right: 0.4em;
}

.quote-box .quote-author {
    width: 450px;
    height: auto;
    clear: both;
    padding-top: 20px;
    font-size: 1em;
    text-align: right;
}

.quote-box .buttons {
    width: 450px;
    margin: auto;
    display: block;
}

.quote-box .buttons .button {
    height: 38px;
    border: none;
    border-radius: 3px;
    color: #fff;
    background-color: #333;
    outline: none;
    font-size: 0.85em;
    padding: 8px 18px 6px 18px;
    margin-top: 30px;
    opacity: 1;
    cursor: pointer;
}

.quote-box .buttons .button:hover {
    opacity: 0.9;
}

.quote-box .buttons .button#tweet-quote,
.quote-box .buttons .button#tumblr-quote {
    float: left;
    padding: 0px;
    padding-top: 8px;
    text-align: center;
    font-size: 1.2em;
    margin-right: 5px;
    height: 30px;
    width: 40px;
}

.quote-box .buttons .button#new-quote {
    float: right;
}
```

**qutoe.js**       

利用了api获得语录，并且在点击时利用,$.animate()完成渐变效果,利用twitter api实现分享功能   

```
  function inIframe() {
      try {
          return window.self !== window.top;
      } catch (e) {
          return true;
      }
  }
  var colors = [
      '#16a085',
      '#27ae60',
      '#2c3e50',
      '#f39c12',
      '#e74c3c',
      '#9b59b6',
      '#FB6964',
      '#342224',
      '#472E32',
      '#BDBB99',
      '#77B1A9',
      '#73A857'
  ];
  var currentQuote = '',
      currentAuthor = '';

  function openURL(url) {
      window.open(url, 'Share', 'width=550, height=400, toolbar=0, scrollbars=1 ,location=0 ,statusbar=0,menubar=0, resizable=0');
  }

  function getQuote() {
      $.ajax({
          headers: {
              'X-Mashape-Key': 'OivH71yd3tmshl9YKzFH7BTzBVRQp1RaKLajsnafgL2aPsfP9V',
              Accept: 'application/json',
              'Content-Type': 'application/x-www-form-urlencoded'
          },
          url: 'https://andruxnet-random-famous-quotes.p.mashape.com/cat=',
          success: function(response) {
              var r = JSON.parse(response);
              currentQuote = r.quote;
              currentAuthor = r.author;
              if (inIframe()) {
                  $('#tweet-quote').attr('href', 'https://twitter.com/intent/tweet?hashtags=quotes&related=freecodecamp&text=' + encodeURIComponent('"' + currentQuote + '" ' + currentAuthor));
              }
              $('.quote-text').animate({ opacity: 0 }, 500, function() {
                  $(this).animate({ opacity: 1 }, 500);
                  $('#text').text(r.quote);
              });
              $('.quote-author').animate({ opacity: 0 }, 500, function() {
                  $(this).animate({ opacity: 1 }, 500);
                  $('#author').html(r.author);
              });
              var color = Math.floor(Math.random() * colors.length);
              $('html body').animate({
                  backgroundColor: colors[color],
                  color: colors[color]
              }, 1000);
              $('.button').animate({ backgroundColor: colors[color] }, 1000);
          }
      });
  }
  $(document).ready(function() {
      getQuote();
      $('#new-quote').on('click', getQuote);
      $('#tweet-quote').on('click', function() {
          if (!inIframe()) {
              openURL('https://twitter.com/intent/tweet?hashtags=quotes&related=freecodecamp&text=' + encodeURIComponent('"' + currentQuote + '" ' + currentAuthor));
          }
      });
      $('#tumblr-quote').on('click', function() {
          if (!inIframe()) {
              openURL('https://www.tumblr.com/widgets/share/tool?posttype=quote&tags=quotes,freecodecamp&caption=' + encodeURIComponent(currentAuthor) + '&content=' + encodeURIComponent(currentQuote));
          }
      });
  });
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p12.png)


**源代码**[RandomQuote](https://github.com/whuhan2013/freeCodeCampProject/tree/master/RandomQuote)

**live Demo**[live Demo](https://whuhan2013.github.io/web/RandomQuote/index.html)






