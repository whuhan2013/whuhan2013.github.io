---
layout: post
title: Portfolio页面实现
date: 2016-10-22
categories: blog
tags: [javascript]
description: Portfolio页面实现
---


**导航页面**

```
<div class="first">
        <!-- START Navigation bar -->
        <div class="top-fixed-nav">
            <div class="top_left">Ricardo.M.Jiang</div>
            <div class="top_right">
                <div>
                    <a href="#home" class="orientbox">Home</a>
                </div>
                <div>
                    <a href="#about" class="orientbox">About</a>
                </div>
                <div>
                    <a href="#portfolio" class="orientbox">Portfolio</a>
                </div>
                <div>
                    <a href="#contact" class="orientbox">Contact</a>
                </div>
            </div>
        </div>
        <!-- END Header -->

         <!-- START Navigation bar -->
        <div class="main-content">
            
            <!-- START Home content -->
            <div id="home" class="home">
                <div class="home-content">
                    <h1>I <strong>LOVE</strong> MYSELF</h1>
                    <h3>love me too!</h3>
                    <div class="social-icons">
                        <a href="https://www.twitter.com/"><i class="fa fa-twitter fa-2x">&nbsp;twitter</i></a>
                        <a href="https://www.facebook.com/"><i class="fa fa-facebook fa-2x">&nbsp;facebook</i></a>
                       <!--  <a href="https://www.linkedin.com/"><i class="fa fa-linkedin fa-2x"></i></a> -->
                        <a href="https://www.github.com/"><i class="fa fa-github fa-2x">&nbsp;github</i></a>
                    </div>
                </div>
            </div>
            <!-- END Home content -->
        </div>
        
    </div>
      <!--END First-->
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p6.png)

**介绍页面** 

```
 <div class="second">
        <div class="about-content">
                    <div class="row">
                        <div class="col-half-left">
                            <h2>About me.</h2>
                            <p>This is actually not me. It is a cat. Or a placeholder. Or you. It depends on a few factors.</p>
                            <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Commodi a consequatur, facere numquam sint earum adipisci aliquam nostrum perspiciatis corporis veritatis repellendus voluptatum voluptas dolores aperiam explicabo obcaecati. Consequuntur, repellendus.</p>
                        </div>
                        <div class="col-half">
                           <div class="photo-rounded">
                               <img src="http://book.img.ireader.com/group6/M00/B3/1F/CmQUNlYB4yCEbgtHAAAAAKOTd38351241604.jpg">
                           </div>
                        </div>
                    </div>
                </div>
    </div>
    <!--END Second-->
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p7.png)


**作品介绍** 

```
<div class="third">
       <div class="third_title">
           <h1>Admire my work</h1>
       </div>
       <div class="third_content">
           <div class="third_col">
               <img src="images/weather.png"/>
               <img src="images/bottom.jpg"/>
               <img src="images/clock.png"/>
           </div>
           <div class="third_col">
               <img src="images/quote.png">
               <img src="images/top.jpg">
               <img src="images/weather.png">
           </div>
       </div>
    </div>
    <!--END Third-->
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p8.png)

**联系方式**

```
    <div class="fourth">
        <!-- START Contact content -->
            
                <div class="contact-content">
                    <div class="row">
                       <h2>Find me pls</h2>
                        <ul>
                            <li>
                                <i class="fa fa-inbox"></i>&nbsp;<a href="mailto:info@gorkahernandez.com">info@gorkahernandez.com</a>
                            </li>
                            <li>
                                <i class="fa fa-phone"></i>&nbsp;1-555-5555
                            </li>
                            <li>
                                <i class="fa fa-building"></i>&nbsp;Awesome St. 103, Power City, NY
                            </li>
                            <li>
                                <i class="fa fa-twitter"></i>&nbsp;<a href="https://www.twitter.com/GorkaJS">GorkaJS</a>
                            </li>
                        </ul>
                    </div>
                </div>
            
            <!-- END Contact content -->
    </div>
    <!--END FOURTH-->
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p9.png)


**源代码:**[portfolio](https://github.com/whuhan2013/freeCodeCampProject/tree/master/portfolio)

