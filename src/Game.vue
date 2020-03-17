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
                    @open="open()">
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
    readonly bombed!: boolean;//これreadonlyだから、爆弾の真偽値Gameの中でいじれないよね？

    status: Status = 'preparing';//初めの状態
    width: number = 5;
    height: number = 5;
    bomb: number = 5;
    opened: number = 0;
    clickCount: number=0;
    setBomb: number=0;
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

    // establish(){
    //     let array=this.getCells();//全てのセルコンポーネントから作成されたものの配列[cells]を取得
        
    // //     for(let q = array.length - 1; q >= 0; q--){//配列の個数から１引いた値。qが０より小さくなるまで、1ずつ減らす　ここ元のコードにはなかったからここら辺も作用してそう
    // //         let cell = array[q];//cells配列のq番目 cellクラスに属する一個一個のcell
    // //         //for (let i = 0; i < this.bomb; i++) {//初期値０であるiに1ずつ足して行って、爆弾の個数の入力値よりもiが小さい限りそれを継続する
    // //             while(this.bombs==this.bomb){//true→cell.bombed=true ここがトゥルーの時にループした＝最後のbreakが機能せず、
    // //                 let a = Math.floor(Math.random() * this.width);//爆弾を入れるセルの座標を決定
    // //                 let b = Math.floor(Math.random() * this.height);
    // //                 if (cell.x==a && cell.y==b && cell.bombed==false) {//a,b座標のcellに爆弾がなければ
    // //                     cell.bombed = true;//cellをx,y座標で区別して、そこに爆弾を配置
    // //                     this.bombs++;
    // //                     //配置できる数値になったら、ループ打ち切ってfor i にもどる
    // //                 }
    // //             }   
    // //         //}
    //     // }
    // }


    establish(){
        let array=this.getCells();
        for (let i = 0; i < this.bomb; i++) {
            let a = Math.floor(Math.random() * this.width);//爆弾を入れるセルの座標を決定
            let b = Math.floor(Math.random() * this.height);
            for(let q = array.length - 1; q >= 0; q--){
                let cell = array[q];
                if (cell.x==a && cell.y==b && !cell.bombed) {//a,b座標のcellに爆弾がなければ
                    cell.bombed = true;//cellをx,y座標で区別して、そこに爆弾を配置
                    this.setBomb++;
                    if(this.bomb==this.setBomb){
                        break;
                    }
                }
            }    
        }    
    }

    //  establish(){
    //     const array=this.getCells();//全てのセルコンポーネントから作成されたものの配列[cells]を取得
    //     for(let q = array.length - 1; q >= 0; q--){//0~qまでの要素に対して
    //         const cell = array[q];//cells配列のq番目 cellクラスに属する一個一個のcell
    //         for (let i = 0; i < this.bomb; i++) {//入力値の数値になるまで爆弾入力の処理を繰り返す
    //             while (true) {//trueのかぎり繰り返し
    //                 let a = Math.floor(Math.random() * this.height);//爆弾を入れるセルの座標を決定
    //                 let b = Math.floor(Math.random() * this.width);
    //                 if (cell.x==a && cell.y==b && !cell.bombed) {//a,b座標のcellに爆弾がなければ
    //                     cell.bombed = true;//cellをx,y座標で区別して、そこに爆弾を配置
    //                     break;//配置したらループ打ち切ってfor i にもどる
    //                 }
    //             }
    //              break;
    //         }
    //     }
    // }

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
        this.status = 'failured';//この後にセルを削除するコードを    ]
        
    }

    reset(){//完成
        this.status = 'preparing';//配置したコンポーネントを動的に取得して、スタイルとかプロパティを取得して弄ったり、処理を実行させたり cellsオブジェクト
        const array=this.getCells();//各cellsコンポーネントが入った、配列　ここでgetcellを呼び出す
        for(let b = array.length - 1; b >= 0; b--){
            const cell = array[b];//b番目のセルをリセットしまくる
            cell.init();
        }
    }

    open(){//成功した場合のゲーム終了関数 仮かんせい opened++でも良さそう
        const array=this.getCells();//全部のcellを取り寄せて、その中から、diggedがtrueのものをopenedにpush 
        for(let b = array.length - 1; b >= 0; b--){
            const cell = array[b];//b番目のセルをリセットしまくる
            if(cell.digged&&cell.bombed==false){
                this.opened++;
                if(this.opened==this.width*this.height-this.bomb){
                    this.status='successed';
                }
            }            
        }
    }
}
</script>
