<template>
    <div class="controls">
        <div class="d-pad" ref="dPad"></div>
        <div class="actions">
            <div class="action" @click="action"></div>
        </div>
        <div class="menu-access" @click="openMenu"></div>
    </div>
</template>

<script>
import nipplejs from 'nipplejs'

const DIRECTIONS = ['left', 'right', 'up', 'down']

export default {
    name: 'rpg-controls',
    inject: ['rpgEngine'],
    mounted() {
        const manager  = nipplejs.create({
            zone: this.$refs.dPad
        })
        let directions = {}
        let moving = false
        manager.on('added', (evt, nipple) => {
            const keyup = (key) => {
                 this.rpgEngine.controls.applyControl(key, false)
            }
            const end = () => {
                moving = false
                for (let key of DIRECTIONS) {
                    keyup(key)
                }
            }
            const move = () => {
                if (moving) {
                    for (let dir in directions) {
                        this.rpgEngine.controls.applyControl(dir, true)
                    }
                }
            }
            nipple.on('end', end)
            nipple.on('move', (evt, data) => {
               if (data.direction) {
                    const degree = data.angle.degree
                    const { x, y, angle } = data.direction
                    directions = {
                        [angle]: true
                    }
                    for (let i = 0 ; i < 4 ; i++) {
                        const corner = 90 * i + 45
                        if (degree < corner + 20 && degree > corner - 20) {
                            directions = {
                                [x]: true,
                                [y]: true
                            }
                        }
                    }

                    for (let dir of DIRECTIONS) {
                        if (!directions[dir]) {
                            keyup(dir)
                        }
                    }
                   
                    moving = true

                    move()
               }
            })
            setInterval(move, 400)
        })
        
    },
    methods: {
        openMenu() {
            this.rpgEngine.controls.applyControl('back')
        },
        action() {
            this.rpgEngine.controls.applyControl('action')
        }
    }
}
</script>

<style scoped>
.controls {
    height: 100%;
    position: absolute;
    width: 100%;
    z-index: 50;
}

.d-pad {
    width: 70%;
    position: absolute;
    height: 100%;
}

.actions {
    width: 30%;
    position: absolute;
    right: 0;
    height: 100%;
}

.action {
    background-image: url('../assets/flatDark35.png');
    width: 80px;
    height: 80px;
    opacity: 0.8;
    bottom: 10px;
    position: absolute;
    right: 10px;
}

.menu-access {
    background-image: url('../assets/flatDark32.png');
    width: 48px;
    height: 48px;
    opacity: 0.8;
    top: 10px;
    position: absolute;
    right: 10px;
}
</style>