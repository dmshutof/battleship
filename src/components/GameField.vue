<template>
    <div>
        <transition name="fade">
            <div v-if="!shipCount" class="game_result">
                <h2 v-if="winGame === 'USER'">Вы победили</h2>
                <h2 v-if="winGame === 'PC'">Вы проиграли</h2>
                <button @click="reset" class="button">начать заного</button>
            </div>
        </transition>
        <div class="field" :class="{disabled: !shipCount,  enemy_field: isEnemy}">
            <div class="field__cell" v-for="(item, index) of gridItems" :key="index"
                 :class="{ship: item.ship /*&& !isEnemy*/, hit: item.hit, dead: item.dead,  miss: item.miss}"
                 @click="isEnemy ? userFire(item) : null">
                <span v-if="item.miss" class="miss">·
                     <img v-if="item.miss" src="../assets/water.gif" alt="" style="width: 30px; height: 37px;">
                </span>
                <img v-if="item.explosion && !item.dead" src="../assets/VJeo.gif" alt=""
                     style="width: 40px; height: 40px;">
                <img v-if="item.dead" src="../assets/fire.gif" alt="" style="width: 30px; height: 37px;">

                <small style="position: absolute; font-size: 8px; left: 1px; bottom: 1px">{{item.coords}}</small>
            </div>
        </div>

    </div>
</template>

<script>
    export default {
        name: "GameField",
        props: {
            isEnemy: Boolean,
            userTurn: Boolean
        },
        data() {
            return {
                gridItems: [],
                shipCount: 0,
                winGame: '',
                optionShip: {
                    count: [1, 2, 3, 4],
                    size: [4, 3, 2, 1]
                },
                ships: [],
                collision: new Set(),
                enemyLastLuckyHit:null
            }
        },
        mounted() {
            this.createGrid()
        },
        methods: {
            userFire(cell) {
                if (!this.shipCount && this.userTurn)
                    return false
                if (cell.hit === false) {
                    cell.miss = true
                    this.$emit('update:userTurn', false)
                    for (let i = 0; i < this.ships.length; i++) {
                        const ship = this.ships[i];
                        const index = ship.location.indexOf(cell.coords);

                        if (index >= 0) {
                            ship.hit[index] = 'x';
                            cell.hit = true
                            cell.miss = false
                            cell.explosion = true
                            setTimeout(() => {
                                cell.explosion = false
                            }, 700)
                            const life = ship.hit.indexOf('');
                            this.$emit('update:userTurn', true)
                            if (life < 0) {
                                let maybeCell = []
                                for (const id of ship.location) {
                                    console.log(id)
                                    this.gridItems.forEach(item => {
                                        if (item.coords === id) {
                                            item.dead = true
                                            let idCoords = {
                                                x: parseInt(id[0]),
                                                y: parseInt(id[1])
                                            }
                                            maybeCell.push((idCoords.x - 1) + '' + idCoords.y,
                                                (idCoords.x + 1) + '' + idCoords.y,
                                                idCoords.x + '' + (idCoords.y - 1),
                                                idCoords.x + '' + (idCoords.y + 1),
                                                (idCoords.x - 1) + '' + (idCoords.y - 1),
                                                (idCoords.x - 1) + '' + (idCoords.y + 1),
                                                (idCoords.x + 1) + '' + (idCoords.y + 1),
                                                (idCoords.x + 1) + '' + (idCoords.y - 1))
                                        }

                                    })
                                }
                                for (const id of maybeCell) {
                                    let item = this.gridItems.find(item => item.coords === id)
                                    if (item) {
                                        !item.dead ? item.miss = true : null
                                    }
                                }


                                this.shipCount -= 1;

                                if (!this.shipCount) {
                                    this.winGame = 'USER'

                                }
                            }
                        }

                    }

                    //если промах - переход хода
                    setTimeout(() => {
                        if (this.userTurn === false) {
                            this.enemyFire()
                        }
                    }, 500)

                }
            },
            enemyFire(enemyCell) {
                let userField = this.$parent.$refs.UserField

                if (!userField.shipCount && !userField.userTurn)
                    return false
                let cell = enemyCell ? enemyCell : userField.gridItems[userField.enemyLastLuckyHit  ? this.getCell(userField,cell) : Math.floor(Math.random() * 100)]
              console.log(enemyCell ,cell.coords)
                if (cell.hit) {
                    if(userField.enemyLastLuckyHit){
                        console.log('userField.enemyLastLuckyHit')
                        this.enemyFire(userField.gridItems[this.getCell(userField,cell)])
                    }else{
                        this.enemyFire()
                    }
                } else {
                    cell.miss = true
                    this.$emit('update:userTurn', true)
                    for (let i = 0; i < userField.ships.length; i++) {
                        const ship = userField.ships[i];
                        const index = ship.location.indexOf(cell.coords);

                        if (index >= 0) {
                            ship.hit[index] = 'x';
                            cell.hit = true
                            cell.miss = false
                            cell.explosion = true
                            userField.enemyLastLuckyHit = cell.coords
                            setTimeout(() => {
                                cell.explosion = false
                            }, 700)
                            const life = ship.hit.indexOf('');
                            this.$emit('update:userTurn', false)
                            setTimeout(() => {
                                this.enemyFire(userField.gridItems[this.getCell(userField,cell)])
                            }, 500)

                            if (life < 0) {
                                let maybeCell = []
                                for (const id of ship.location) {
                                    userField.gridItems.forEach(item => {
                                        if (item.coords === id) {
                                            item.dead = true
                                            userField.enemyLastLuckyHit = null
                                            let idCoords = {
                                                x: parseInt(id[0]),
                                                y: parseInt(id[1])
                                            }
                                            maybeCell.push((idCoords.x - 1) + '' + idCoords.y,
                                                (idCoords.x + 1) + '' + idCoords.y,
                                                idCoords.x + '' + (idCoords.y - 1),
                                                idCoords.x + '' + (idCoords.y + 1),
                                                (idCoords.x - 1) + '' + (idCoords.y - 1),
                                                (idCoords.x - 1) + '' + (idCoords.y + 1),
                                                (idCoords.x + 1) + '' + (idCoords.y + 1),
                                                (idCoords.x + 1) + '' + (idCoords.y - 1))
                                        }

                                    })
                                }
                                for (const id of maybeCell) {
                                    let item = userField.gridItems.find(item => item.coords === id)
                                    if (item) {
                                        !item.dead ? item.miss = true : null
                                    }
                                }

                                userField.shipCount -= 1;

                                if (!userField.shipCount) {
                                    this.winGame = 'PC'

                                }
                            }
                        }

                    }
                }

            },
            getCell(userField, cell){
                let coords = userField.enemyLastLuckyHit || cell.coords,
                    hitCell = userField.gridItems.find(item => item.coords === coords),
                    maybeCell = []
                maybeCell.push((hitCell.x - 1) + '' + hitCell.y,
                    (hitCell.x + 1) + '' + hitCell.y,
                    hitCell.x + '' + (hitCell.y - 1),
                    hitCell.x + '' + (hitCell.y + 1))


               let notHitItems = []
                for (const id of maybeCell) {
                    let item = userField.gridItems.find(item => item.coords === id && !item.hit && !item.miss)
                    console.log('item',item)
                    if (item) {
                        notHitItems.push(id)

                    }
                }
                console.log(notHitItems)
               return notHitItems[Math.floor(Math.random() * notHitItems.length)]

            },
            createGrid() {
                let index = 0;
                for (let x = 0; x < 10; x++) {
                    for (let y = 0; y < 10; y++) {
                        this.gridItems.push(
                            {
                                x,
                                y,
                                coords: x + '' + y,
                                index: index,
                                miss: false,
                                hit: false,
                                dead: false,
                                ship: false,
                                explosion: false,
                            }
                        );
                        index++;
                    }
                }
                this.generateShip()
            },
            generateShip() {
                for (let i = 0; i < this.optionShip.count.length; i++) {
                    for (let j = 0; j < this.optionShip.count[i]; j++) {
                        const size = this.optionShip.size[i];
                        const ship = this.generateOptionsShip(size);
                        this.ships.push(ship);
                        this.shipCount++;
                    }
                }
                this.ships.forEach(ship => {
                    this.gridItems.forEach(item => {
                        ship.location.forEach(location => {
                            if (item.coords === location) {
                                item.ship = true
                            }
                        })
                    })
                })
            },
            generateOptionsShip(shipSize) {
                const ship = {
                    hit: [],
                    location: [],
                };

                const direction = Math.random() < 0.5;
                let x, y;

                if (direction) {
                    x = Math.floor(Math.random() * 10);
                    y = Math.floor(Math.random() * (10 - shipSize));
                } else {
                    x = Math.floor(Math.random() * (10 - shipSize));
                    y = Math.floor(Math.random() * 10);
                }

                for (let i = 0; i < shipSize; i++) {
                    if (direction) {
                        ship.location.push(x + '' + (y + i))
                    } else {
                        ship.location.push((x + i) + '' + y)
                    }
                    ship.hit.push('');
                }

                if (this.checkCollision(ship.location)) {
                    return this.generateOptionsShip(shipSize);
                }
                this.addColission(ship.location);
                return ship;
            },
            checkCollision(location) {
                for (const coord of location) {
                    if (this.collision.has(coord)) {
                        return true;
                    }
                }
            },
            addColission(location) {
                for (let i = 0; i < location.length; i++) {
                    const startCoordX = location[i][0] - 1;

                    for (let j = startCoordX; j < startCoordX + 3; j++) {
                        const startCoordY = location[i][1] - 1;

                        for (let z = startCoordY; z < startCoordY + 3; z++) {

                            if (j >= 0 && j < 10 && z >= 0 && z < 10) {
                                const coord = j + '' + z;

                                this.collision.add(coord);
                            }


                        }
                    }

                }
            },
            reset() {
                this.ships = []
                this.gridItems = []
                this.winGame = ''
                this.collision = new Set()
                this.userTurn = true
                this.shipCount = 0
                this.createGrid()
                let userField = this.$parent.$children.find(item => item._uid === 3)
                userField.ships = []
                userField.gridItems = []
                userField.winGame = ''
                userField.collision = new Set()
                userField.userTurn = true
                userField.shipCount = 0
                userField.createGrid()
            }
        }
    }
</script>

<style scoped lang="scss">

    .field {
        display: grid;
        grid-template-columns: repeat(10, 40px);
        border: 1px solid #b4b4ff;

        &.disabled {
            pointer-events: none;
        }
    }

    .enemy_field {
        .field__cell {
            pointer-events: all;
            position: relative;
            &:hover {
                &:before{
                    content: '';
                    position: absolute;
                    width: 100%;
                    height: 100%;
                    border: 3px solid #de2414;
                    left: -3px;
                    top: -3px;
                    z-index: 1;
                }
            }
        }
    }

    .field__cell {
        height: 40px;
        width: 40px;
        border: 2px solid #ffffff;
        cursor: pointer;
        display: flex;
        justify-content: center;
        align-items: center;
        pointer-events: none;
position: relative;

        &.ship {
            background: green;
        }

        &.miss {
            pointer-events: none;
            animation: bg_fade .5s .3s ease-in-out forwards;
            background: #ffffff;
        }
        @keyframes bg_fade {
            100% {
                background: transparent;
            }
        }
        &.hit {
            background: yellow;
            pointer-events: none;
        }

        &.dead {
            background: darkred;
        }

        .miss {
            font-size: 64px;
            img{
                position: absolute;
                top: 50%;
                left: 50%;
                transform: translate(-50%,-50%);
                animation: hide 1s .3s ease-in-out forwards;
            }
        }
    }

    .game_result {
        position: fixed;
        background: rgba(0, 0, 0, 0.8);
        width: 100%;
        height: 100%;
        left: 0;
        top: 0;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;

        h2 {
            color: #ffffff;
            font-size: 38px;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: -2.48px;
            text-align: center;
            margin-bottom: 50px;
        }

        img {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            animation: hide 1s .3s ease-in-out forwards;
            pointer-events: none;
        }


    }
    @keyframes hide {
        100% {
            opacity: 0;
        }
    }
    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s;
    }

    .fade-enter, .fade-leave-to /* .fade-leave-active до версии 2.1.8 */
    {
        opacity: 0;
    }

</style>