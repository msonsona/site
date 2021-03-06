---
layout: post
title:  "Playing in your browser"
categories: gamedev
tags: html5 javascript
excerpt: In which the author builds a first version of an html5 pong game
---

**tl;dr** You can [play pong](/uploads/playing-in-your-browser/tennis.html)!

Since being little, when playing around building with LEGO or Playmobil, I've been fascinated by creating new worlds and adventures. Growing up, one usually switch to play videogames (until having a baby, where you can go back to play with your old toys :smile:). However, playing videogames mostly isn't about creating or designing experiences, with some exceptions.

Developing games combines some interesting features like:

- providing an interactive experience, 
- creating a universe / world / ambient (which can be a mix of artistic work and procedurally generation), 
- designing intelligent opponents or other characters 

and more, so really at some point I though I had to give it a try. But it also requires some new knowledge.

One of the easiest ways of starting off with a new technology is following a tutorial or a getting started guide. Additionally, nowadays there are other options like online courses (MOOCs) which provide a more engaging learning experience. In this case, I found a free course on [Udemy](https://www.udemy.com) about learning game programming in html5. A couple of comments about Udemy:

Nice:

- Offline video lessons on the app (with download over wifi). Very useful for me, as I usually watch them while commuting. 

Meh:

- The web in general, and the course in particular, don't seem to work properly on Chrome, then I just have to fallback to Safari. 

Anyway, there is literally nothing to loose, apart of a little time, which is always an investment when learning new things. Let's go!

The course is [Code Your First Game: Arcade Classic in JavaScript on Canvas](https://www.udemy.com/code-your-first-game/) by [Chris DeLeon](http://gamkedo.com). It is a good starting point, the author is clear and to the point, gives appropriate background introducing new topics (the course is for beginners) and makes the course interesting and engaging.

Upon finishing the course, you end up having a fully playable version of pong like [this one](/uploads/playing-in-your-browser/original-tennis.html), where you move the left paddle and the computer moves the right paddle, until either of the players reach 3 points to win the game.

With some [research](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Drawing_text) on how to change the font used to display text in the canvas, we can obtain something a bit more [classic-looking](/uploads/playing-in-your-browser/styled-tennis.html):
![Improving the style](/uploads/playing-in-your-browser/tennis.png)

The game [code](https://github.com/msonsona/html5-games/blob/master/tennis.html) (which is all JavaScript) has some key parts, like the initialization of the canvas element (where we'll literally draw the game) and the creation of the update / draw cycle that composes the game engine. We also capture the mouse position to move the left player paddle and add a listener function to the mouse click event to start a new game:

{% highlight javascript %}
window.onload = function() {
    // initialize the canvas element where we'll draw the game
    canvas = document.getElementById("gameCanvas");
    canvasContext = canvas.getContext('2d');
    
    // define the engine's update - draw cycle at 30 fps
    var framesPerSecond = 30;
    setInterval(function() {
        moveEverything();
        drawEverything();
    }, 1000/framesPerSecond);
    
    // update left player paddle position capturing mouse movements
    canvas.addEventListener('mousemove', 
        function(evt) {
            var mousePos = calculateMousePosition(evt);
            paddle1Y = mousePos.y - PADDLE_HEIGHT/2;
        });
    
    // calling function on mouse click, to resume game after victory
    canvas.addEventListener('mousedown', handleMouseClick);
}
{% endhighlight %}

The most crucial part of the game is where all the positions are computed:

{% highlight javascript %}
function computerMovement() {
    var paddle2YCenter = paddle2Y + PADDLE_HEIGHT/2;
    
    // move the computer player's paddle if it's not well aligned with the ball
    if (paddle2YCenter < ballY - 35) {
        paddle2Y += 6;
    } else if (paddle2YCenter > ballY + 35) {
        paddle2Y -= 6;
    }
}

function moveEverything() {
    computerMovement();
    
    // update ball position based on current position + 2D speed
    ballX = ballX + ballSpeedX;
    ballY = ballY + ballSpeedY;
    
    // collision detection on the left edge of the screen
    if (ballX < 0) {
        if (ballY > paddle1Y
            && ballY < paddle1Y + PADDLE_HEIGHT) {
            // the paddle hit the ball, make it go the other way
            ballSpeedX = -ballSpeedX;
            
            // affect direction and speed depending on the part of the paddle
            var deltaY = ballY - (paddle1Y + PADDLE_HEIGHT/2);
            ballSpeedY = deltaY * 0.35;
        } else { // the player didn't hit the ball
            player2Score++; // update score before reset
            ballReset();
        }
    }
    // collision detection on the right edge of the screen
    if (ballX > canvas.width) {
        if (ballY > paddle2Y
            && ballY < paddle2Y + PADDLE_HEIGHT) {
            // the paddle hit the ball, make it go the other way
            ballSpeedX = -ballSpeedX;
            
            // affect direction and speed depending on the part of the paddle
            var deltaY = ballY - (paddle2Y + PADDLE_HEIGHT/2);
            ballSpeedY = deltaY * 0.35;
        } else { // the player didn't hit the ball
            player1Score++; // update score before reset
            ballReset();
        }
    }
    // collision detection on the top edge of the screen
    if (ballY < 0) {
        // rebound the ball vertically
        ballSpeedY = -ballSpeedY;
    }
    // collision detection on the bottom edge of the screen
    if (ballY > canvas.height) {
        // rebound the ball vertically
        ballSpeedY = -ballSpeedY;
    }
}
{% endhighlight %}

And then the function that draws all the game elements. Here the important parts are the positioning of both the ball and the players' paddles, just on the positions we computed earlier in the `moveEverything` function.

{% highlight javascript %}
function drawEverything() {
    // screen background
    colorRect(0,0,canvas.width,canvas.height,'black');
    
    drawNet();
    
    // player paddle
    colorRect(0,paddle1Y,PADDLE_WIDTH,PADDLE_HEIGHT,'white');
    // computer paddle
    colorRect(canvas.width-PADDLE_WIDTH,paddle2Y,PADDLE_WIDTH,PADDLE_HEIGHT,'white');
    
    // ball
    colorCircle(ballX, ballY, 10, 'white');
    
    // players' scores
    canvasContext.font = "40px monospace";
    canvasContext.fillText(player1Score, 100, 100);
    canvasContext.fillText(player2Score, canvas.width-100, 100);
}
{% endhighlight %}

You can check the full [source](https://github.com/msonsona/html5-games/blob/master/tennis.html) or directly [play the game](/uploads/playing-in-your-browser/tennis.html). As usual, there is room for improvement, new features to add and explorations on the technology to make the game more interesting, so stay tuned!

P.S.: The author of the course on Udemy has another (paid) course that extends on the free one I took, [How to Program Games: Tile Classics in JS for HTML5 Canvas](https://www.udemy.com/how-to-program-games/), and taking advantage of a discount I decided to purchase it, so more things to learn, play with and write about!