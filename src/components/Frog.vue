<!--***********************************************************************************************************************************************
    HTML/TEMPLATE AREA
 ***********************************************************************************************************************************************-->
<template>
    <div>
        <br>
        <div class="points_container">
            <div class="points_style"><!-- PLAYER POINTS -->
                <img src="../assets/fly.png" width="30px" height="30px">
                {{player.points}}
            </div>
            <div class="flies_eaten_style"><!-- JUMP EATEN FLIES SCORE-->
                <img src="../assets/arc.png"  width="23px" height="23px">
                <img src="../assets/frog.png" width="35px" height="30px" class="frog_little">
                <img src="../assets/fly.png"  width="20px" height="20px" style="transform: scaleX(-1)">
                {{frog.jump.collisions}}
            </div>
            <div class="flies_max_eaten_style"><!-- JUMP MAX FLIES EATEN SCORE-->
                <img src="../assets/arc.png"  width="23px" height="23px">
                <img src="../assets/frog.png" width="35px" height="30px" class="frog_little">
                <img src="../assets/fly.png"  width="20px" height="20px" style="transform: scaleX(-1)">
                <img src="../assets/fly.png"  width="20px" height="20px" style="transform: scaleX(-1)">
                <img src="../assets/fly.png"  width="20px" height="20px" style="transform: scaleX(-1)">
                <img src="../assets/fly.png"  width="20px" height="20px" style="transform: scaleX(-1)">
                {{frog.jump.collisions_max}}
            </div>
        </div>
        <img src="../assets/frog.png" :style="frog.style"> <!-- FROG IMAGE -->
        <img src="../assets/fly.png"  :style="newfly.style" v-for="newfly of fly.list" :key="newfly.position"> <!--FLIES IMAGES -->
    </div>
</template>

<!--***********************************************************************************************************************************************
    JAVASCRIPT AREA 
***********************************************************************************************************************************************-->
<script>
export default {
    name: 'Frog',
    data: function() { // MAIN DATA
        return {
            game: {    // GENERAL GAME DATA
                window_width : window.innerWidth, 
                window_height: window.innerHeight,
                sounds: {
                    soundtrack        : this.load_sound('soundtrack.mp3'),
                    soundtrack_started: false // TO AVOID THE "user didn't interact with the document first" ISSUE, THE MUSIC STARTS AT THE FIRST CLICK
                }
            },
            player: { // PLAYER DATA
                points: 0 // NUMBER OF FLIES EATEN
            },
            frog: { // FROG DEFINITION
                style: {
                    position : 'absolute',
                    left     : '3px',
                    top      : '0px',
                    width    : '188px',
                    height   : '155px',
                    transform: 'scaleX(-1)'
                },
                jump: {
                    going_left    : false,                   // HELPS TO KNOW WHICH WAY THE NEXT JUMP HAS TO BE
                    jumping       : false,                   // HELPS TO KNOW IF THE FROG IS ALREADY JUMPING >> TO MANAGE MULTIPLE PLAYER CLICKS
                    angle         : Math.PI/180,             // FROG'S JUMP UNIQUE ANGLE
                    radius        : 650,                     // FROG'S JUMP RADIUS
                    center_x      : 650,                     // FROG'S JUMP'S CENTER X COORDINATE
                    center_y      : window.innerHeight - 250,// FROG'S JUMP'S CENTER Y COORDINATE
                    collisions    : 0,                       // FROG'S COLLISIONS PER JUMP
                    collisions_max: 0                        // FROG'S MAX COLLISIONS PER JUMP
                },
                sounds: {
                    jump: this.load_sound('jump.wav'),
                    eat:  this.load_sound('eat.wav')
                }
            },
            fly: { // FLIES MAIN DATA
                list:       [], // ARRAY OF LIVING FLIES
                current_max: 5, // MAX LIVING FLIES
                maximun:    20, // TOP MAX LIVING FLIES
                speed:       1, // BASE SPEED OF EVERY FLY. THIS NUMBERS GROWS BY 1 EVERY 20 POINTS
                max_step:    3, // MAX SPEED OF EVERY FLY. **** TODO: THIS MUST BE GROW SOMEHOW WITH THE speed PROPERTY ****
            }
        }
    },
    /*************************************************************************************************************************************************
    *   METHODS AREA
    *************************************************************************************************************************************************/    
    created: function() {
        this.game_loop()
    },
    methods: {
        /*************************************************************************************************************************************************
         *  game_loop() GAME MAIN LOOP
        *************************************************************************************************************************************************/
        game_loop: function() {            
            window.onclick                     = this.player_click                      // LINKS player_click() WITH window.onclick
            this.frog.style.top                = (this.game.window_height - 250) + 'px' // FROG'S INITIAL POSITION
            this.game.sounds.soundtrack.volume = 0.5
            setInterval(() => {
                this.generate_fly() // PERMANENTLY GENERATES NEW FLIES UNTIL IT REACHES fly.max
            }, 1)
        },
        /*************************************************************************************************************************************************
         *  player_click() THIS METHOD IS TRIGGERED EVERY TIME THE PLAYER RIGHT CLICKS THE BROWSER
        *************************************************************************************************************************************************/
        player_click: function(event) {
            // TO AVOID THE "user didn't interact with the document first" ISSUE, THE MUSIC STARTS AT THE FIRST PLAYER CLICK
            if(!this.game.sounds.soundtrack_started) {
                this.game.sounds.soundtrack.play()
                this.game.sounds.soundtrack_started = true
            }
            if(event.button == 0) 
                this.frog_jump() // IF PLAYER RIGHT-CLICKED
        },
        /*************************************************************************************************************************************************
         *  frog_jump() THIS FUNCTION DOES THE FROG'S JUMP TO BOTH DIRECTIONS AND CALLS THE check_collisions() METHOD
        *************************************************************************************************************************************************/
        frog_jump: function() {
            if(this.frog.jump.jumping) 
                return
            this.frog.sounds.jump.play()
            this.frog.jump.collisions = 0
            this.frog.jump.jumping    = true
            let degrees               = (this.frog.jump.going_left)? 360 : 180
            let degrees_limit         = (this.frog.jump.going_left)? 180 : 360
            let id = setInterval( () => {
                let new_point         = this.generate_point(degrees)
                this.frog_move_to(new_point.x, new_point.y)
                this.frog.jump.collisions += this.check_collisions()
                degrees += (this.frog.jump.going_left)? -1 : 1
                if(degrees == degrees_limit) {
                    clearInterval(id)
                    this.frog.style.transform = (this.frog.jump.going_left)? 'scaleX(-1)' : 'scaleX(1)'
                    this.frog.jump.jumping    = false
                    this.frog.jump.going_left = !this.frog.jump.going_left
                    if(this.frog.jump.collisions > this.frog.jump.collisions_max)
                        this.frog.jump.collisions_max = this.frog.jump.collisions
                    setTimeout(() => {
                        if(!this.frog.jump.jumping) {
                            this.frog.jump.collisions = 0
                        }
                    }, 1500);
                }
            }, 10)
        },
        frog_move_to: function(x, y) {
            this.frog.style.left = x
            this.frog.style.top  = y
        },
        /*************************************************************************************************************************************************
         *  generate_point() GENERATES A NEW POINT FOR THE FROG'S JUMP
        *************************************************************************************************************************************************/
        generate_point: function(degree) {
            var x = this.frog.jump.center_x + ( this.frog.jump.radius * Math.cos(degree * this.frog.jump.angle) )
            var y = this.frog.jump.center_x + ( this.frog.jump.radius * Math.sin(degree * this.frog.jump.angle) )
            return {x: x + 'px', y: y + 'px'}
        },
        /*************************************************************************************************************************************************
         *  generate_fly() CREATES A NEW FLY ONLY IF THE fly.max IS NOT REACHED YET
         *                 ON TOP OF THAT, KEEPS MOVING THE FLY LEFT TO RIGHT (AND VICEVERSA) AND TOP TO BOTTOM (SLIGHTLY)
        *************************************************************************************************************************************************/
        generate_fly: function() {
            if(this.fly.list.length < this.fly.current_max) {
                let fly_new = {
                    style: {
                        position: 'absolute',
                        left: '10px',
                        top:  (50 + Math.floor(Math.random() * (this.game.window_height / 2))) + 'px',
                        width: '50px',
                        height: '52px',
                        transform: 'scaleX(1)'
                    },
                    left_way: false,
                    step: this.generate_fly_step()
                }
                if(this.generate_left_fly()) {
                    fly_new.left_way        = true
                    fly_new.style.left      = (parseInt(this.game.window_width) - 101) + 'px'
                    fly_new.style.transform = 'scaleX(-1)'
                }
                this.fly.list.push(fly_new)
                setInterval( () => {
                    let step           = fly_new.step * (fly_new.left_way ? -1 : 1)
                    let fly_new_left   = parseFloat(fly_new.style.left) + step
                    fly_new.style.left = fly_new_left + 'px'
                    
                    let move_top = this.move_fly_top()
                    let fly_new_top = parseFloat(fly_new.style.top) + ((move_top)? (step * -1) : step)
                    if(fly_new_top > parseInt(fly_new.style.height) && fly_new_top < (this.game.window_height - 250 - parseInt(fly_new.style.height))) {
                        fly_new.style.top = fly_new_top + 'px'
                    }
                    if(!fly_new.left_way && fly_new_left >= (this.game.window_width - 100)) {
                        fly_new.left_way = true
                        fly_new.style.transform = 'scaleX(-1)'
                    } else if(fly_new.left_way && fly_new_left < 30) {
                        fly_new.left_way = false
                        fly_new.style.transform = 'scaleX(1)'
                    }
                }, this.fly.speed)
            }
        },
        /*************************************************************************************************************************************************
         *  generate_fast_fly() RETURNS A BOOLEAN THAT IS USED TO GENERATE A FAST FLY (OR A SLOW ONE INSTEAD)
        *************************************************************************************************************************************************/
        generate_fast_fly: function() {
            return (Math.random() > 0.55)
        },
        /*************************************************************************************************************************************************
         *  generate_left_fly() RETURNS A BOOLEAN THAT IS USED TO GENERATE A FLY THAT SHOWS FROM THE LEFT (OR RIGHT INSTEAD)
        *************************************************************************************************************************************************/
        generate_left_fly: function() {
            return (Math.random() > 0.5)
        },
        /*************************************************************************************************************************************************
         *  generate_left_fly() RETURNS A BOOLEAN THAT IS USED TO MOVE A FLY TO THE TOP DIRECTION (OR BOTTOM INSTEAD)
        *************************************************************************************************************************************************/
        move_fly_top: function() {
            return (Math.random() > 0.5)
        },
        /*************************************************************************************************************************************************
         *  generate_fly_step() RETURNS THE SPEED FOR A NEW FLY. CAN BE FAST OR SLOW DEPENDING ON generate_fast_fly()
        *************************************************************************************************************************************************/
        generate_fly_step: function() {
            if(this.generate_fast_fly()) {
                return Math.random() * this.fly.max_step
            } else {
                return 0.1 * this.fly.max_step
            }
        },
        /*************************************************************************************************************************************************
         *  get_entity_hitbox() RETURNS A HIT-BOX FOR THE FROG OR A FLY
         * 
         *  x1, y1 ---------- x2, y1
         *    |                 |
         *    |                 |
         *    |                 |
         *  x1, y2 ---------- x2, y2
         *    
         * 
        *************************************************************************************************************************************************/
        get_entity_hitbox: function(entity) {
            return {
                x1: parseInt(entity.style.left),
                x2: parseInt(entity.style.left) + parseInt(entity.style.width),
                y1: parseInt(entity.style.top),
                y2: parseInt(entity.style.top) + parseInt(entity.style.height)
            }
        },
        /*************************************************************************************************************************************************
        *  check_collisions() CHECK THE FROG'S HITBOX AGAINST EVERY LIVING FLY.
        *                     ON ANY COLLISION THE FLY 
        *************************************************************************************************************************************************/
        check_collisions: function() {
            let frog_box = this.get_entity_hitbox(this.frog), collision
            for(let fly of this.fly.list) {
                collision = this.check_collision(frog_box, this.get_entity_hitbox(fly))
                if(collision) {
                    this.fly.list.splice(this.fly.list.indexOf(fly), 1)
                    this.frog.sounds.eat.play()
                    this.player.points++
                    if(this.player.points % 25 == 0) { //EVERY 50 POINTS, 1 MORE FLY IS ALLOWED TO APPEAR
                        if(this.fly.current_max < this.fly.maximum) {
                            this.fly.current_max++
                        }
                    }
                    return 1
                }
            }
            return 0
        },
        check_collision: function(frog_box, fly_box) {
            // COLLISION 1: LEFT SIDE OF THE FROG
            if(frog_box.x1 >= fly_box.x1 && frog_box.x1 <= fly_box.x2) {
                if(frog_box.y1 >= fly_box.y1 && frog_box.y1 <= fly_box.y2) {
                    return true //TOP LEFT COLLISION
                }
                if(frog_box.y2 >= fly_box.y1 && frog_box.y2 <= fly_box.y2) {
                    return true //CENTER LEFT COLLISION
                }
                if(frog_box.y1 <= fly_box.y1 && frog_box.y2 >= fly_box.y2) {
                    return true //BOTTOM LEFT COLLISION
                }
            }
            //COLLISION 2: CENTER SIDE OF THE FROG
            if(frog_box.x1 <= fly_box.x1 && frog_box.x2 >= fly_box.x2) {
                if(frog_box.y1 >= fly_box.y1 && frog_box.y1 <= fly_box.y2) {
                    return true //TOP CENTER COLLISION
                }
                if(frog_box.y2 >= fly_box.y1 && frog_box.y2 <= fly_box.y2) {
                    return true //BOTTOM CENTER COLLISION
                }
            }
            //COLLISION 3: RIGHT SIDE OF THE FROG
            if(frog_box.x2 >= fly_box.x1 && frog_box.x2 <= fly_box.x2) {
                if(frog_box.y1 >= fly_box.y1 && frog_box.y1 <= fly_box.y2) {
                    return true //TOP RIGHT COLLISION
                }
                if(frog_box.y1 <= fly_box.y1 && frog_box.y2 >= fly_box.y2) {
                    return true //CENTER RIGHT COLLISION
                }
                if(frog_box.y2 >= fly_box.y1 && frog_box.y2 <= fly_box.y2) {
                    return true //BOTTOM RIGHT COLLISION
                }
            }
        },
        load_sound: function(name) {
            if(name.substring(name.length-3) == 'mp3') {
                return new Audio(require.context('../assets/sounds/', false, /\.mp3$/)('./' + name))
            } else {
                return new Audio(require.context('../assets/sounds/', false, /\.wav$/)('./' + name))
            }
        }
    }
}
</script>
<!------------------------------------------------------------------------------------------------------------------------------------ 
    CSS/STYLE AREA
------------------------------------------------------------------------------------------------------------------------------------>
<style scoped>

    .points_container {
        display        : flex;
        justify-content: space-around;
        /* border         : 1px solid; */
    }

    .points_style {
        font-size       : 50px;
        font-weight     : bolder;
        color           : rgb(255, 255, 255);
        background-color: rgba(0, 140, 255, 0.8);
        width           : 250px;
        border-radius   : 20px;
        text-align      : center;
    }

    .flies_eaten_style, .flies_max_eaten_style {
        color           : rgb(0,150,50);
        width           : 200px;
        text-align      : center;
        font-size       : 50px;
        border-radius   : 20px;
        font-weight     : bolder;
        background-color: rgba(251, 255, 0, 0.8);
    }
    .flies_max_eaten_style {
        color: red;
    }
    .frog_little {
        transform: scaleX(-1);
    }

</style>

<!------------------------------------------------------------------------------------------------------------------------------------ 
    END OF FILE
 ------------------------------------------------------------------------------------------------------------------------------------>