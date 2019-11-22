<style lang="scss">
* {
    box-sizing: border-box;
    padding: 0;
    margin: 0;
    border: none;
    box-shadow: none;
    background: none;
}
</style>

<style scoped lang="scss">
main {
    h1 {
        padding: 0.2em 0.5em;
    }
    input[type="number"]:not(:disabled){
        border: solid 1px black;
    }
    button{
        background-color: #aaa;
        padding: 0.5em;
        cursor: pointer;
        border-radius: 0.2em;
    }
}
</style>

<template>
<main>
    <h1>{{title}}</h1>
    <div>
        <p>
            <input type="number" v-model.number="width" :min="3" :max="30" :disabled="status != 'preparing'" />
            x
            <input type="number" v-model.number="height" :min="3" :max="30" :disabled="status != 'preparing'" />
        </p>
        <p>
            💣：<input type="number" v-model.number="bomb" :min="1" :max="width * height" :disabled="status != 'preparing'" />
        </p>
        <button v-if="status == 'preparing'" @click="start">Start</button>
        <button v-else-if="status == 'playing'" @click="giveup">Giveup</button>
        <button v-else @click="reset">Reset</button>
    </div>
    <table>
        <tr v-for="y in height" :key="y">
            <td v-for="x in width" :key="x">
                <cell
                    ref="cells"
                    :status="status"
                    :arounds="gatherAroundCells(x, y)"
                    :x="x"
                    :y="y"
                    @update="recount()">
                </cell>
            </td>
        </tr>
    </table>
</main>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";
import Cell from "./Cell.vue"; //vueの導入のための記述

export type Status = 'preparing'|'playing'|'successed'|'failured'; //statusによってゲームの状態を規程

const components = {
    'cell': Cell
};

@Component({components})//コンポーネントの宣言
export default class Game extends Vue{ //Gameと言うクラススタイルVueコンポーネントを宣言
    status: Status = 'preparing';//初めの状態
    width: number = 5;
    height: number = 5;
    bomb: number = 5;

    created(){
        document.title = 'Game'; //タイトル設定
    }

    get title(): string{//ゲッター　プロパティが参照された時に
        return "Title";
    }

    private getCells(): Cell[]{
        const cells = this.$refs.cells;//配置したコンポーネントを動的に取得して、スタイルとかプロパティを取得して弄ったり、処理を実行させたりコンポーネントインスタンスcells
        if(cells instanceof Array) return cells as Cell[];
        if(cells instanceof Cell) return [cells];
        return [];
    }

    gatherAroundCells(x: number, y: number){
        return this.getCells().filter(cell=>{//取得したcellに対して以下の要件に合致するものだけを取り出す
            if(cell.x == x && cell.y == y) return false;//周辺のcellだから
            if(cell.x < x - 1) return false;//枠外のcellをカウントしないための条件
            if(cell.x > x + 1) return false;
            if(cell.y < y - 1) return false;
            if(cell.y > y + 1) return false;
            return true;
        });
    }

    recount(){//なんの関数か？
        
    }

    start(){
        this.status = 'playing';
    }

    giveup(){
        this.status = 'failured';
    }

    reset(){
        this.status = 'preparing';
        this.width= 5;
        this.height= 5;
        this.bomb= 5;
    }

}
</script>
