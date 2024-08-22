# GSAP 


## gsap.from()

- The gsap.from() method is used to animate elements from a new state to their current state.
Meaning animate `from animation to current/defined state`

    ```js
    gsap.from("#box1", 
        {duration: 5,
        x: 500,
        y: 400, 
        rotation: 360,
        opacity: 0, 
        scale: 0.1,
        ease: "elastic"
        });
    ```
    - `x`: Value in x-axis from which you want animation to start
    - `y`: Value in x-axis from which you want animation to start
    - `rotation`: used to rotate the object
    - `opacity`: set the Transparency from `0` to `1`
    - `scale`: set the size of the object from `0.1` to `1`

## gsap.to()

- The gsap.to() method is used to animate elements from current state to new state.
Meaning animate `from current to animated state`

    ```js
    gsap.to("#box1", 
        {duration: 5,
        x: 500,
        y: 400, 
        rotation: 360,
        scale: 0.1,
        ease: "elastic"
        });
    ```
    - `x`: Value in x-axis to which you want animation to take your object in
    - `y`: Value in x-axis from which you want animation to take your object in
    - `rotation`: used to rotate the object
    - `scale`: set the size of the object from `1` to `0.1`



## gsap.fromTo()

- The gsap.fromTo() method is used to animate elements from x state to y state.
Meaning animate `from current to animated state`

    ```js
    gsap.fromTo("#box2", 
    {x: 500, opacity: 0, scale: 0.5},
    {duration: 5, x: 300, opacity: 1, scale: 1,ease: "strong.inOut"}
    );
    ```
    > In above box2 will start `from` `x-500,opacity-0,size-0.5` `to` `x-300,opacity-1,size-1`

## gasp.timeline()

-  Helps you organize multiple animations into a sequence. 

    ```js
    var t1 = gsap.timeline();

    t1.from("#box1", {duration: 3, x: 500,y:400, rotation: 360, scale: 0.1,ease: "strong.inOut"});
    t1.to("#box3", {y: 700,duration:2, opacity: 1, scale: 1.5});
    t1.to("#box2", {x: 500,duration:2, opacity: 1, scale: 0.5});
    ```
    >This will start animation in sequence `box1`-->`box3`---> `box2`

- To add default properties for all:suppose duration for all be 3

    ```js
    var t1 = gsap.timeline({ defaults: { duration: 3} });
    ```

## Gsap staggers

- In simple words, GSAP staggers allow you to animate multiple elements in a sequence, one after another, with a delay in between each animation. Instead of all elements starting at the same time, staggers help you create a cascading effect where each element starts slightly after the previous one.
    ```js
    gsap.from(".box", {duration: 3,x:100,ease: "back.in",stagger: 1 ,opacity:0});
    ```
    >All elements in .box(Class box can have multiple elements) will follow one after another animation with `stagger`

## GSAP ScrollTrigger
- Helps to add animation to any element whenever it is visisble in screen
    ```js
    let tl = gsap.timeline({
        scrollTrigger: {
            trigger: "#page2 .container",
            scroller: "#",
            markers: true,
            start: "top 50%",
            scrub: 3,
        }
    });

    // Add animations to the timeline
    tl.to("#page2 .container .box1", { xPercent: -17, duration: 4 })
      .to("#page2 .container .box2", { xPercent: 20, duration: 4 }, "<") // "<" starts this animation at the same time as the previous one
      .to("#page2 .container .box3", { xPercent: -17, duration: 4 }, "<");
    ```
    >Note: I want all `box1,box2,box3` to have translation in same scroll which is `#page2 .container`
    
- For Individual use like this:  
    ```js
    gsap.to(
        "#page2 .container .box1",
        {
            transform:"translateX(-17%)",
            duration:4,
            scrollTrigger:{
                trigger:"#page2 .container ",
                scroller:"body",
                markers:true,
                start:"top 50%",
                scrub:3,
            }
        }
    )

    ```
    > `scrollTrigger` is property for scroll trigger
    >
    > `trigger` is the trigger in which we want to activate the trigger.In above in `#page2 .container` we want to trigger
    >
    >`scroller` is mostly body
    >
    >`markers` it is for development purpose to show the trigger line of both screen and element
    >
    >`start` it sets the trigger to top 50% of `scroller` which is `body`
    >
    >`end` it sets the trigger to end defined of `scroller` which is `body`
    >
    >`scrub` it `sets the animation on basis of scrolling` like the more we scroll the animation is done and if we scroll up animation is `undo` or `reverse`  

- Below is the html code for the above 
    ```html
    <body>
        <div id="page1">
            <h1>Learning gsap</h1>
        </div>
        <div id="page2">
            <div class="container">
                <div class="box1">
                    <p>Hello1</p>
                    <p>Hello2</p>
                    <p>Hello3</p>
                    <p>Hello4</p>
                    <p>Hello5</p>
                    <p>Hello6</p>
                    <p>Hello7</p>
                    <p>Hello8</p>
                    <p>Hello9</p>
                    <p>Hello10</p>
                    <p>Hello11</p>
                </div>
                <div class="box2">
                    <p>Suman1</p>
                    <p>Suman2</p>
                    <p>Suman3</p>
                    <p>Suman4</p>
                    <p>Suman5</p>
                    <p>Suman6</p>
                    <p>Suman7</p>
                    <p>Suman8</p>
                    <p>Suman9</p>
                    <p>Suman10</p>
                    <p>Suman11</p>
                </div>
                <div class="box3">
                    <p>Bhola1</p>
                    <p>Bhola2</p>
                    <p>Bhola3</p>
                    <p>Bhola4</p>
                    <p>Bhola5</p>
                    <p>Bhola6</p>
                    <p>Bhola7</p>
                    <p>Bhola8</p>
                    <p>Bhola9</p>
                    <p>Bhola10</p>
                    <p>Bhola11</p>
                </div>
            </div>

        </div>
        <div id="page3">

        </div>

        <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js" ></script>
        <script src="app.js"></script>

    </body>
    ```
    and css
    ```css
    * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    }
    body{
        overflow-x: hidden;
    }

    #page1{
        height: 100vh;
        width: 100%;
        background-color: #fc7676;
        display: flex;
        justify-content: center;
        align-items: center;
    }
    #page2{
        height: 100vh;
        width: 100%;
        background-color: #76fc76;
        display: flex;
        justify-content: center;
        align-items: center;
    }
    #page3{
        height: 100vh;
        width: 100%;
        background-color: #6c84f4;
    }
    .container{
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        width: 100%;
        background-color: #2f2f2f;
        padding-top: 30px;
        padding-bottom: 30px;
    }
    .box1 {
        display: flex;
        gap: 30px;
    }
    .box2 {
        display: flex;
        gap: 30px;
    }
    .box3 {
        display: flex;
        gap: 30px;
    }
    .box2 p{
        color: red;
        font-size: 60px;
    }

    .box3 p{
        color: blue;
        font-size: 60px;
    }
    p{
        color: white;
        font-size: 60px;
    }
    ```

## GSAP Text Effect

- dwad
    ```js
    const h1=document.querySelector("h1");
    const h1Text=h1.textContent;

    const splittedChars = h1Text.split(""); 

    h1.innerHTML = '';

    splittedChars.forEach(char => {
        const span = document.createElement('span');
        span.textContent = char;
        h1.appendChild(span);
    });

    gsap.from("h1 span", {
        yPercent: 130,
        stagger:0.05,
        ease:"back.out(1.7)",
        duration:1,
        scrollTrigger:{
            trigger:".container ",
            scroller:"body",
            markers:true,
            start:"top 40%",
            scrub:2
        }
    }
    )
    ```
    >Here we first select text in h1,split it and append it in h1 as each char have span...now we do `gsap.from("h1 span")`

    >Remember to add css to span `h1 span {
    >display: inline-block; /* Allows transforms to work */}`