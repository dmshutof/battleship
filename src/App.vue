<template>
    <div id="app">

        <div class="bg" :class="{dark:start_game }">
            <transition name="fade">
                <div v-if="!start_game">
                    <h1>Морской бой</h1>
                    <div class="button" @click="step = 2" v-if="step === 1">Играть</div>
                    <form @submit.prevent="StartGame" v-if="step === 2">
                        <div class="input">
                            <label for="i1">Ваше имя</label>
                            <input autocomplete="off" id="i1" type="text" placeholder="Введите ваше имя" v-model="name"
                                   required>
                        </div>
                        <div class="input">
                            <label for="i2">Имя противника</label>
                            <input id="i2" type="text" placeholder="Введите имя противника" v-model="enemy_name"
                                   required>
                        </div>
                        <button type="submit" class="button">Начать игру</button>
                    </form>
                </div>
                <Battleground v-else :name="name" :enemy-name="enemy_name"></Battleground>
            </transition>
        </div>

    </div>
</template>

<script>
    import Battleground from "./components/Battleground";

    export default {
        name: 'App',
        components: {
            Battleground
        },
        data() {
            return {
                name: '',
                enemy_name: '',
                start_game: false,
                step: 1
            }
        },
        methods: {
            StartGame() {
                if (!!this.name && !!this.enemy_name) {
                    this.start_game = true
                }
            }
        }
    }
</script>

<style lang="scss">
    @import "css/reset.css";
    @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

    * {
        padding: 0;
        margin: 0;
        box-sizing: border-box;
        font-family: 'Roboto', sans-serif;
    }

    h1 {
        font-size: 48px;
        font-weight: 700;
        color: #fff;
        text-transform: uppercase;
        letter-spacing: -2.48px;
        text-align: center;
        margin-bottom: 50px;
    }

    .bg {
        min-height: 100vh;
        padding: 5vw 2.5vw;
        background: url(https://wows-wowsp-global.gcdn.co/media/ceph-image/1bb1e2ae-ff4b-11eb-8ee6-8cdcd4b147d4.jpg) no-repeat center;
        background-size: cover;
        display: flex;
        justify-content: center;
        align-items: center;
        text-align: center;
        flex-direction: column;
        transition: background .3s ease-in-out;
        z-index: 1;

        &.dark {
            /*&:after{
                content: '';
                pointer-events: none;
                background: rgba(0,0,0,.6);
                position: absolute;
                left: 0;
                top: 0;
                bottom: 0;
                right: 0;
                z-index: 0;
            }*/
        }
    }

    .button {
        cursor: pointer;
        text-decoration: none;
        border: none;
        display: flex;
        justify-content: center;
        align-items: center;
        text-align: center;
        padding: 12px 30px;
        font-size: 18px;
        font-weight: 700;
        line-height: 24px;
        text-transform: uppercase;
        background-image: -webkit-gradient(linear, left bottom, left top, from(#de2414), to(#fc611d));
        background-image: linear-gradient(to top, #de2414 0%, #fc611d 100%);
        background-size: 100% 100%;
        color: #fff;
        -webkit-transition: all 0.3s;
        transition: all 0.3s;

        &:hover {
            opacity: .8;
        }

        &:disabled {
            background-image: linear-gradient(to top, #363636 0%, #7e7c7b 100%);
            pointer-events: none;
        }
    }

    form {
        display: grid;
        grid-gap: 16px;
        width: 100%;
        max-width: 340px;

        label {
            color: #ffffff;
            margin-bottom: 5px;
            text-align: left;
            font-size: 14px;
        }

        .input {
            display: grid;
            width: 100%;
        }

        input {
            width: 100%;
            border: none;
            height: 46px;
            padding: 5px 10px;

        }
    }
</style>
