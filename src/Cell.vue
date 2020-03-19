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
    <div :class="{digged}" @click="dig()" @contextmenu.prevent="mark()"> <!--v-on,v-bindの省略記法-->
        {{display()}}
    </div>
    
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";//TypeScriptでコンポーネントが書けるようになる。 Componentデコレータをつけて、Vueを継承したクラスとして書く
import {Status} from './Game.vue';

@Component({
    props: ['status', 'arounds', 'x', 'y','width','height','count'] 
})//GameからCellに　続けて定義しているクラスをVueが認識できる形式に変換

export default class Cell extends Vue{
    readonly status!:Status;//　!はnull/undefinedではないことを意味している
    readonly arounds!: Cell[];
    readonly x!: number;
    readonly y!: number;
    readonly width!: number;
    readonly array3!: Cell[];
    readonly height!: number;
    readonly bomb!: number;
    readonly count!: number;
    bombed: boolean = false;
    digged: boolean = false;
    marked: boolean = false;
    check: number=0;

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
        let array = this.arounds.filter(element => element.bombed==true).length;//ボムがある周りのます自体をarrayに代入　そしてarrayの数を出力する
        return array;
    }

    display(){//マスに表示するマーク、爆弾、周辺の爆弾の数
        if(this.marked) return '🚩';
        if(this.digged){//!外した
            if(this.bombed==true) return '💥';
            return this.aroundBombsNumber || '';
        }
        return '';
    }

    dig(){
        if(!this.mutable) return;//undifinedが帰る
        if(this.marked) return;
        this.check++
        this.$emit('edit');
        let indicator: number=0;
        if(this.bombed==true&&this.array3.length==1){//カウント１の時は再配置
            this.bombed=false;
            this.digged = true;
            while(indicator!=1){
                let a = Math.floor(Math.random() * this.width +1);
                let b = Math.floor(Math.random() * this.height +1);
                if(a==this.x&&b==this.y){//クリックされたx,yと同じ座標のcellだったら他の座標が生成され、array2の長さがbombと一致するまでは繰り返し、それ以外のますで、爆弾がなければ設置する
                }else{
                    this.$emit('put',{x:a , y:b})
                    indicator=indicator++;
                }
            }
        }else if(this.bombed==true&&this.array3.length!=1){//ここはOK　カウント２以上で爆弾掘った時の処置
            this.digged = true;
            this.$emit('update');
        }else{//爆弾がない時の処置
            if(this.arounds.length==0){
                this.digged = true;
                this.$emit('open')
                this.$emit('hatch',{x:this.x,y:this.y})
            }else{
                this.digged = true;
                this.$emit('open')
            }
        }
    }

    mark(){
        if(!this.mutable) return;
        this.marked = !this.marked;
    }
}
</script>
