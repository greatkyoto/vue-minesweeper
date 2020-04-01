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
        <button v-if="status == 'preparing'" @click="start">Start</button><!--v-ifで表示を切り替え-->
        <button v-else-if="status == 'playing'" @click="recount">Giveup</button>
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
                    :array3="array3"
                    @update="recount()"
                    @put="put(x,y)"
                    @open="open()"
                    @edit="edit()"
                    @hatch="hatched(x,y)">
                </cell><!-- refでrefsに登録　: bindの省略　＠v-onの省略 :bombs="gatherAroundBombs(x,y)"-->
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
    components: {Cell}
})
export default class Game extends Vue{ //Gameと言うクラススタイルVueコンポーネントを宣言
    digged!: boolean;//!はnull/undefinedではないことを意味している
    bombed!: boolean;
    bombs!: number
    status: Status = 'preparing';//初めの状態
    width: number = 5;
    height: number = 5;
    bomb: number = 5;
    opened: number = 0;
    array3: Cell[]=[];
    
    
    created(){//void型　つまり何も返さない　ライフサイクルフック　インスタンス作成されたら
        document.title = 'Game'; //タイトル設定
        let array=this.getCells();
        
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

    gatherAroundBombs(x: number, y: number){//ただ周りのマスを集めた物を返している
        return this.getCells().filter(cell=>{//getCells()で全てのcellを習得　filter関数で
            if(cell.x == x && cell.y == y) return false;//周辺のcellだから
            if(cell.x < x - 1) return false;//枠外のcellをカウントしないための条件
            if(cell.x > x + 1) return false;
            if(cell.y < y - 1) return false;
            if(cell.y > y + 1) return false;
            if(cell.bombed==false) return false;//周辺のcellだから
            return true;
        }).length;
    }


    hatched(x: number,y: number){//周辺に爆弾がない場合の、周辺の爆弾の処理
        for (let i = x - 1; i <= x + 1; i++) {//iがx-1から これだとx,y入るけど他の条件で弾ける １＝＜this.x＝＜this.width　外のマスはないとは思うが、念のため１以上の処理記述
            for (let j = y - 1; j <= y + 1; j++) {
                if(i　>= 1 && j >= 1){
                    const cell=this.getCells().find(element => element.x==i && element.y==j && element.digged==false && element.bombed==false && ( element.x != x || element.y != y ));//全てのセルの中から
                    if(cell!=undefined){
                        if (cell.bombs==0){//の条件を満たすものを配列に仕立てる　ここでbombsが０のものに関してはhatch(i,j)したらおk
                            cell.digged = true;
                            this.open();//この後の処理として、周辺にばくだんがないものの処理　this.$refs.cells. 開ける処理をしたらもう一回こっちに戻ってきてそのx,yをここに入れたい
                            this.hatched(i,j);
                        }else{
                            cell.digged = true;
                            this.open();
                        }
                    }
                }
            }
        }
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
    
    edit(){//一番初めのマスカどうかのチェック
        let array=this.getCells();
        for(let b = array.length - 1; b >= 0; b--){
            const cell = array[b];
            if(cell.checked==true){//クリックされた回数が1回のものは
                this.array3.push(cell);//array３配列に追加　  
            }
        }
    }
    

    recount(){//ゲーム終了　全てのセルオープン　完成
        const array=this.getCells();
        for(let b = array.length - 1; b >= 0; b--){
            const cell = array[b];//b番目のセルをリセットしまくる
            cell.digged=true;
            cell.marked=false;
        }
        this.giveUp();
    }

    start(){//完成
        if(this.bomb!=null && this.height!=null && this.width!=null){
            if (this.bomb > 0 && this.height > 0 && this.width > 0 && this.bomb < this.width * this.height) {
                this.status = 'playing';
                this.establish();
                let array=this.getCells();
                for(let b = array.length - 1; b >= 0; b--){
                    const cell = array[b];
                    cell.bombs=this.gatherAroundBombs(cell.x,cell.y);
                }
            }else{
                alert("入力値はルールに則ってください");
            }
        }else{
            alert("全て入力してください");
        }
    }

    giveUp(){//完成
        this.status = 'failured';//この後にセルを削除するコードを    
        //alert("Faulse")
    }

    reset(){//完成
        this.array3.length=0;
        this.opened=0;
        this.status = 'preparing';//配置したコンポーネントを動的に取得して、スタイルとかプロパティを取得して弄ったり、処理を実行させたり cellsオブジェクト
        const array=this.getCells();//各cellsコンポーネントが入った、配列　ここでgetcellを呼び出す
        for(let b = array.length - 1; b >= 0; b--){
            const cell = array[b];
            cell.init();
        }
    }

    open(){//成功した場合のゲーム終了関数 成功パターンの終了指標管理
        this.opened++;
        if(this.opened==((this.width*this.height)-this.bomb)){
            this.status='successed';
            this.array3.length=0;
            this.opened=0;
            alert("Congratulations")
        }
    } 


    put(x:number,y:number){//作成した座標にマッチするセルにボム
        let array=this.getCells();
        let indicator: number=0;
        while(indicator!=1){
            let a = Math.floor(Math.random() * this.width +1);
            let b = Math.floor(Math.random() * this.height +1);//a,bは生成されている
            if( a==x && b==y ){
            }else{
                let cell = array.find(element=>element.x==a && element.y==b) //そこに爆弾がなければ
                if(cell!=undefined){
                    if(cell.bombed==false){
                        cell.bombed=true;
                        
                        indicator=indicator+1;
                    }else{
                    }
                }
            }
        }
    }

       

}
</script>
