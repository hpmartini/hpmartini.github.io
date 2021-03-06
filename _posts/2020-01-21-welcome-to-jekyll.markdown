---
layout: post
title:  "Parallax directive in Angular 8"
date:   2020-01-21 10:34:14 +0100
categories: jekyll update
---
Parallax effects on Single Page Applications are definitely an old hat. However there's always a first time to use something. After testing a lot out, which didn't work like I needed it, I came up with a solution, based on a [blog entry](https://medium.com/fove/angular-parallax-d1c2de9f07a6) by [Gaurav Foujdar](https://medium.com/@gauravkumarfoujdar). But with a different way to move the elements.

# TLDR
Implement the parallax effect with an Angular directive, which works on the translate-y property of CSS3.

See the [demo on Stackblitz](https://stackblitz.com/edit/angular-8-parallax-directive) or [check out from GitHub](https://github.com/hpmartini/angular-8-parallax-directive).

# The parallax effect...
... not explained. I won't bother you with boring explainations about that effect, which you can read for yourself on [wikipedia](https://en.wikipedia.org/wiki/Parallax) if you feel the urge.

# Prerequisites
First you have to have `node.js` running on your system. 

### For Mac users
If you're lucky to have a Mac, just run this:

{% highlight bash %}
brew install node
{% endhighlight %}

If you don't have `homebrew` installed already, go to [homebrew.sh](https://homebrew.sh) or run this:

{% highlight bash %}
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
{% endhighlight %}

### For anyone else
Sorry for that, but please visit [nodejs.org](https://nodejs.org/) for further instructions. I don't have another system at hand right now.

## Ahead with angular cli
Install Angulars command line interface with this, if you dont't have it already:

{% highlight bash %}
npm install -g @angular/cli
{% endhighlight %}

## Setup the Angular project
Go to the projects directory of your desire and run this command to create an Angular project:
{% highlight bash %}
ng new "my-awsome-parallax-project"
{% endhighlight %}

This creats your awesome parallax project in the directory called... I guess you know it already. The CLI fills up the project with everything needed to get it running. So start it up:

{% highlight bash %}
ng serve -o
{% endhighlight %}

The "-o" is not recommended, it just opens the angular app in your default browser.

# Now the fun part starts
Lets generate a new directive:
{% highlight bash %}
ng generate directive common/parallax
{% endhighlight %}

This command puts the new directive in the sub directory `common`, which is not necessary, but I like to have a clean structure.

The CLI also registers the directive in the `app.modules.ts`, along with the default components and services.

### parallax.directive.ts [view on GitHub](https://github.com/hpmartini/angular-8-parallax-directive/blob/master/src/app/common/parallax.directive.ts)
{% highlight TypeScript %}
import {Directive, ElementRef, HostListener, Input, Renderer2} from '@angular/core';

@Directive({
  selector: '[appParallax]'
})
export class ParallaxDirective {
  @Input('factor') set parallaxFactor(val) {
    this.factor = val ? val : 1;
  }

  private factor: number;

  constructor(
    private elementRef: ElementRef,
    private renderer: Renderer2
  ) { }

  @HostListener('window:scroll')
  onWindowScroll() {
    this.renderer.setProperty(
      this.elementRef.nativeElement, 
      'style',
      `transform: translateY(${this.getTranslation()}px)`);
  }

  private getTranslation() {
    return window.scrollY * this.factor / 10;
  }
}
{% endhighlight %}

- The `factor` of how fast the element will move relative to the scroll position is passed through an `@Input` decorator. 
  By default (factor = 1) the effect will be just a bitt slower then a linear scrolling element. 
- The class `ElementRef` is the recommended way to manipulate elementes, because directly accessing the DOM could promote XSS attacs.
- The `Renderer2` takes the `ElementRef` and sets the desired property in the DOM.
- We just set the `translateY` CSS property of our element by multiplying the current scroll position with the passed `factor`. 
  I devide that by 10, because the movement would be to fast or we could not work with integers in the HTML.
- The `@HostListener('window:scroll')` will be triggered when you scroll - what a surprise - and the `translate` propery will be updated.

### app.component.html [view on GitHub](https://github.com/hpmartini/angular-8-parallax-directive/blob/master/src/app/app.component.html)
{% highlight html %}
<div class="container">
  <hello name="{{ name }}"></hello>
  <p appParallax [factor]="11">I'm slowly inverting.</p>
  <p appParallax [factor]="12">I'm doing it faster!</p>
  <p appParallax [factor]="13">Me even more.</p>
  <p appParallax [factor]="14">And I rock!</p>
</div>

<div class="container">
  <h1 appParallax [factor]="0.3">Hello Parallax</h1>

  <div id="linearScrollingText">I just scroll in linear speed</div>

  <div id="slowText" appParallax [factor]="9">I'm going faaaa... not.</div>
</div>
{% endhighlight %}

### app.component.css [view on GitHub](https://github.com/hpmartini/angular-8-parallax-directive/blob/master/src/app/app.component.css)
{% highlight css %}
.container {
  height: 100vh;
}

#slowText {
  position: absolute;
  bottom: -30px;
  right: 10px;
}

#linearScrollingText {
  position: absolute;
  bottom: 10px;
  right: 10px;
}
{% endhighlight %}

Nothing special here, the `container` is set to `100vh` so we can scoll at all. The same goes for the positioning of the two strings at the bottom.

That's it.

Just to remember, you can watch the [demo on Stackblitz](https://angular-8-parallax-directive.stackblitz.io/) or [check out from GitHub](https://github.com/hpmartini/angular-8-parallax-directive).

