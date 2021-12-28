<template>
    <div>
        <h2 v-if="userTurn">Ваш ход</h2>
        <h2 v-else>Ход противника</h2>


        <div class="battleground">
            <div>
                <div class="field_label">
                    Ваше поле({{name}})
                </div>
                <GameField :is-enemy="false" :user-turn.sync="userTurn" ref="UserField"
                          ></GameField>
            </div>
            <div>
                <div class="field_label">
                    Поле вашего противника({{enemyName}})
                </div>
                <GameField :is-enemy="true" :user-turn.sync="userTurn" ></GameField>
            </div>
        </div>
    </div>
</template>

<script>
    import GameField from "./GameField";

    export default {
        name: "Battleground",
        props: ['name', 'enemyName'],
        components: {
            GameField
        },
        data() {
            return {
                userTurn: true
            }
        }
    }
</script>

<style scoped lang="scss">
    .battleground {
        display: grid;
        grid-template-columns: 400px 400px;
        grid-gap: 50px;
        justify-items: center;
        justify-content: center;

        div {
            max-width: 400px;
        }
        @media(max-width: 480px){
            grid-template-columns: 300px;
            div {
                max-width: 400px;
            }
        }
    }

    * {
        color: #fff;
    }

    h2 {
        color: #ffffff;
        font-size: 38px;
        font-weight: 700;
        text-transform: uppercase;
        letter-spacing: -2.48px;
        text-align: center;
        margin-bottom: 50px;
    }

    .field_label {
        margin-bottom: 10px;
    }
</style>