<style scoped lang="scss">
    div {
        cursor: pointer;

        width: 1.5em;
        height: 1.5em;

        background-color: #ccc;
        &.digged {
            background-color: #666;
        }
    }
</style>

<template>
    <div :class="{digged}" @click="dig()" @contextmenu.prevent="mark()"> <!--v-on,v-bindの省略記法 クラスは知らんけど、他は行けてる-->
        {{display()}}<!--旗は出る-->
    </div>
    
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";//TypeScriptでコンポーネントが書けるようになる。 Componentデコレータをつけて、Vueを継承したクラスとして書く
import {Status} from './Game.vue';

@Component({
    props: ['status', 'arounds', 'x', 'y'] //'bomb','width','height']
})//GameからCellに　続けて定義しているクラスをVueが認識できる形式に変換

export default class Cell extends Vue{
    readonly status!:Status;//data
    readonly arounds!: Cell[];
    readonly x!: number;
    readonly y!: number;
    // readonly bomb!: number;//これだとセルが持ってることになっちゃうから、やっぱgameで？
    // readonly width!: number;
    // readonly height!: number;

    bombed: boolean = false;
    digged: boolean = false;
    marked: boolean = false;

    // beforeCreate(){
    //     this.digged=false;
    // }

    init(){
        this.bombed=false;
        this.digged=false;
        this.marked=false;
    }

    get mutable(){//get = computed のこと statusを取得するためのメソッド
        if(this.status != 'playing') return false;//プレイ中じゃなきゃfalseで実行できない
        if(this.digged) return false;//掘ってても実行できない
        return true;
    }

    get aroundBombsNumber(){//周辺の爆弾の数を返す物　//gatherAroundCellsをGameから取得
        var array = this.arounds.filter(element => element.bombed=true).length;//ボムがある周りのます自体をarrayに代入　そしてarrayの数を出力する
        return array;
    }

    // establish(x: number, y: number){//それぞれのセルごとで行われるから入力よりも多くなってしまう
    //     for (var i = 0; i < this.bomb; i++) {//ここはこのまま
    //         while (true) {//ここもこのまま
    //             var a = Math.floor(Math.random() * this.height);//このa,bの座標を
    //             var b = Math.floor(Math.random() * this.width);
    //             if (!this.bombed) {//ここに入れたい
    //                 this.bombed = true;
    //                 break;
    //             }
    //         }
    //     }
    // }

    display(){//マスに表示するマーク、爆弾、周辺の爆弾の数
        if(this.marked) return '🚩';//機能している
        if(this.digged){//!外した
            if(this.bombed) return '💥';//機能してない　爆弾が配置されていないから
            return this.aroundBombsNumber || '';
        }
        return '';
    }

    dig(){
        if(!this.mutable) return;//undifinedが帰る
        if(this.marked) return;
        this.digged = true;
        if(this.bombed){
            this.$emit('update'); 
        }//CellからGameに
    }

    mark(){
        if(!this.mutable) return;
        this.marked = !this.marked;
    }


}
</script>
