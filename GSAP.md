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