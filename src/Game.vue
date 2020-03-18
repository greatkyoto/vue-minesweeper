<style lang="scss">
* {
    box-sizing: border-box;
    padding: 0;
    margin: 0　auto;
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
        <h2>ルール</h2>
        <ol>
            <li>マス数の入力値は、３〜３０の数値にしてください</li>
            <li>爆弾数の入力値は、１以上で、マス数の縦横の積以下にしてください</li>
        </ol>
        <p>
            <input type="number" v-model.number="width" :min="3" :max="30" :disabled="status != 'preparing'" />
            x
            <input type="number" v-model.number="height" :min="3" :max="30" :disabled="status != 'preparing'" />
        </p>
        <p>
            💣：<input type="number" v-model.number="bomb" :min="1" :max="width * height" :disabled="status != 'preparing'" />
        </p>
        <button v-if="status == 'preparing'" v-on:click="start()">Start</button>
        <button v-else-if="status == 'playing'" @click="recount">Giveup</button>
        <button v-else @click="reset">Reset</button>
    </div>
    <table id="table">
        <tr v-for="y in height" :key="y"><!--指定した回数表示を繰り返す　v-bind:key="y"-->
            <td v-for="x in width" :key="x">
                <cell
                    ref="cells"
                    :status="status"
                    :arounds="gatherAroundCells(x, y)"
                    :x="x"
                    :y="y"
                    @update="recount()"
                    @open="open()"
                    @edit="edit()"
                    @collect="getCells()">
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

@Component({
    components: {Cell}//これって書く必要あるのか？
})
export default class Game extends Vue{ //Gameと言うクラススタイルVueコンポーネントを宣言
    readonly digged!: boolean;//!はnull/undefinedではないことを意味している
    bombed!: boolean;//これreadonlyだから、爆弾の真偽値Gameの中でいじれないよね？
    
    status: Status = 'preparing';//初めの状態
    width: number = 5;
    height: number = 5;
    bomb: number = 5;
    opened: number = 0;
    clickCount: number=0;
    
    created(){//void型　つまり何も返さない　ライフサイクルフック　インスタンス作成されたら
        document.title = 'Game'; //タイトル設定
    }
    get title(): string{//ゲッター　参照のみ　statusの状態で書き換え
        return "MineSweeper";
    }

    private getCells(): Cell[]{//cellを全て取得して、配列の中に
        const cells = this.$refs.cells;//配置したコンポーネントを動的に取得して、スタイルとかプロパティを取得して弄ったり、処理を実行させたりコンポーネントインスタンスcells
        if(cells instanceof Array) return cells as Cell[];//cellsがArrayクラスに属する場合は　ダウンキャスト
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


    establish(){
        let array=this.getCells();
        let array2: Cell[]=[];
        while(array2.length!=this.bomb){//爆弾の置く個数分ランダム生成は必要
            let a = Math.floor(Math.random() * this.width +1);
            let b = Math.floor(Math.random() * this.height +1);
            let cell=array.find(element=>element.x==a&&element.y==b)
            if(cell!=undefined){
                if(cell.bombed==false){
                    cell.bombed=true;
                    array2.push(cell);
                }
            }  
        }
    }
    
    edit(){
        let array=this.getCells();
        for(let b = array.length - 1; b >= 0; b--){
            const cell = array[b];
            cell.bombed=false;
        }//一旦ここで切って
    }
    
    collect(){
        let array=this.getCells();
    }

    recount(){//ゲーム終了　全てのセルオープン　完成
        const array=this.getCells();
        for(let b = array.length - 1; b >= 0; b--){
            const cell = array[b];//b番目のセルをリセットしまくる
            cell.digged=true;
            cell.marked=false;
        }
        this.giveup();
    }


    start(){//完成
        if(this.bomb!=null && this.height!=null && this.width!=null){
            if (this.bomb > 0 && this.height > 0 && this.width > 0 && this.bomb < this.width * this.height) {
                this.status = 'playing';
                this.establish();
            }else{
                alert("入力値はルールに則ってください");
            }
        }else{
            alert("全て入力してください");
        }
    }

    giveup(){//完成
        this.status = 'failured';//この後にセルを削除するコードを    
    }

    reset(){//完成
        this.clickCount=0;//clickCountをゼロに
        this.status = 'preparing';//配置したコンポーネントを動的に取得して、スタイルとかプロパティを取得して弄ったり、処理を実行させたり cellsオブジェクト
        const array=this.getCells();//各cellsコンポーネントが入った、配列　ここでgetcellを呼び出す
        for(let b = array.length - 1; b >= 0; b--){
            const cell = array[b];//b番目のセルをリセットしまくる
            cell.init();
        }
    }

    open(){//成功した場合のゲーム終了関数 仮かんせい opened++でも良さそう あとは爆弾がない隣接したマスをを一緒に開ける
        const array=this.getCells();//全部のcellを取り寄せて、その中から、diggedがtrueのものをopenedにpush 
        for(let b = array.length - 1; b >= 0; b--){
            const cell = array[b];
            if(cell.digged&&cell.bombed==false){
                if(cell.aroundBombsNumber==0){
                    cell.dig();
                }
                this.opened++;//これ書き換え必要
                if(this.opened==(this.width*this.height-this.bomb)){
                    this.status='successed';
                }
            }            
        }
    }


}
</script>
