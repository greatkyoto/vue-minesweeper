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
    <h1>{{title}}</h1><!--computedの呼び出しだから（）ない-->
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
        <tr v-for="y in height" :key="y"><!--指定した回数表示を繰り返す　v-bind:key="y"-->
            <td v-for="x in width" :key="x">
                <cell
                    ref="cells"
                    :status="status"
                    :arounds="gatherAroundCells(x, y)"
                    :x="x"
                    :y="y"
                    @update="recount()"
                    @gather="gatherAroundCells()"><!--aroundsはただ単に近接した８マスを集めた物-->
                </cell><!--cellタグはcellコンポーネントから-->
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

@Component({
    components: {Cell}//これって書く必要あるのか？
})
export default class Game extends Vue{ //Gameと言うクラススタイルVueコンポーネントを宣言
    readonly digged!: boolean;

    status: Status = 'preparing';//初めの状態
    width: number = 5;
    height: number = 5;
    bomb: number = 5;
    opened: number = 0;

    created(){
        document.title = 'Game'; //タイトル設定
    }

    

    get title(): string{//ゲッター　参照のみ　statusの状態で書き換え
        return "MineSweeper";
    }

    private getCells(): Cell[]{//cellを全て取得して、配列の中に
        const cells = this.$refs.cells;//配置したコンポーネントを動的に取得して、スタイルとかプロパティを取得して弄ったり、処理を実行させたりコンポーネントインスタンスcells
        if(cells instanceof Array) return cells as Cell[];//cellsがArrayクラスに属する場合は
        if(cells instanceof Cell) return [cells];//cellsがCellクラスに属する場合は
        return [];
    }

    gatherAroundCells(x: number, y: number){//ただ周りのマスを集めた物を返している
        return this.getCells().filter(cell=>{//getCells()で全てのcellを習得　filter関数で
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

    bombEastablish(){
        var saibous: any = this.$refs.cells;
        saibous.establish();//establish自体に問題があるわけではなく、インポート自体に問題がある
    }

    start(){
        //ここでestablish関数を呼び出して、爆弾をランダム配置したいy　refsを使用して、実装可能
        this.status = 'playing';
    }

    giveup(){
        this.status = 'failured';
        
    }

    reset(){
        this.status = 'preparing';
        const cells = this.$refs.cells;
        //cells.digged= 'false'　代案：全てのcellの真偽値を
        //ここに前回のますを削除する
    }

    finish(){//このほかに開けたますをopenedに追加する関数
        if(this.opened==this.width*this.height){
            this.status='successed';
            
        }
    }

}
</script>
