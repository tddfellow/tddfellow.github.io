---
layout: post
title: "HighScore Kata"
description: "Today we will take a look into a little problem involving high scores in some sort of game. The game has only one high score and when current game's score exceeds that number, it gets updated. Example acceptance test would read like this..."
date: 2016-09-30 18:23:21 +0200
comments: true
categories:
- javascript
- kata
- clean-architecture
- clean-code
- testing
---

Hello, everyone. Today we will take a look into a little problem involving high scores in some sort of game. The game has only one high score and when current game's score exceeds that number, it gets updated. Example acceptance test would read like this:

**Given** high score is `174`  
**When** player scores `191`  
**Then** high score is `191`  

<!--more-->

Current implementation stores high score in the web browser's local storage. This detail does not change the purpose of this Kata very much since any other platform and language can have its own analog of local storage (file system, in-memory or local database, application settings, etc.). `HighScore` object looks like this:

```javascript
function Highscore() {
    // initially, load high score value from the local storage
    this.load();
}

Highscore.prototype.updateHighscore = function (score) {
    // check if we need to update high score
    if (score > this.highscore) {
        this.highscore = score;
        this.save();
    }

    // render the high score in the
    // id="highscore" element in the browser
    $("#highscore").text("HIGHSCORE: " + this.highscore);
};

Highscore.prototype.load = function () {
    // localStorage is storing everything as strings,
    // so we need to convert it to number
    this.highscore = parseFloat(localStorage.highscore);
};

Highscore.prototype.save = function () {
    localStorage.highscore = this.highscore;
};
```

Task for the Kata:

- Write tests for this class.
- Fix the bug: when a game is launched on the new client (without the high score stored), it renders `HIGHSCORE: NaN`. (`NaN` is javascript's abbreviation for "not a number"). `parseFloat` most probably is a culprit for this.
- Extract storing mechanisms, so that class can be re-used with different storage mechanisms (for example local database, or external REST API).
- Make all tests runnable outside of the context of the browser (for example, on `nodejs`).

The focus of this Kata is on architectural boundaries, that this little innocent class spans.

Questions to ask yourself:

- How much distinct responsibilities this class has?
- What architectural boundaries should I draw through this class?
  - How do I split this class according to these boundaries?
- What is the easiest way and what is the proper way to make tests runnable outside of the context of the browser?
- How would this code look like in different kinds of languages? (static, dynamic, object-oriented, functional, strong-typed, etc.).
  - And how the solution for the Kata will look like for these?

Next time we will take a look at one possible solution for this Kata. Try to solve it on your own, my dear reader, and please share the code and insights!

## Thanks

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@tdd_fellow](https://twitter.com/tdd_fellow).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@tdd_fellow](https://twitter.com/tdd_fellow).
