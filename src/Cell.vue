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
    <div :class="{digged}"
         @click="dig()"
         @contextmenu.prevent="mark()"> <!--v-on,v-bindの省略記法　:bombs="aroundBombsNumber"-->
        {{display()}}
    </div>
    
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";//TypeScriptでコンポーネントが書けるようになる。 Componentデコレータをつけて、Vueを継承したクラスとして書く
import Game, {Status} from './Game.vue';

@Component({
    props: ['status', 'arounds', 'x', 'y','width','height','array3'] 
})//GameからCellに　続けて定義しているクラスをVueが認識できる形式に変換

export default class Cell extends Vue{
    readonly status!:Status;//　!はnull/undefinedではないことを意味している
    readonly arounds!: Cell[];
    readonly x!: number;
    readonly y!: number;
    readonly width!: number;
    readonly height!: number;
    readonly array3!: Cell[];
    bombed: boolean = false;
    digged: boolean = false;
    marked: boolean = false;
    checked: boolean = false;
    bombs: number = 0; //0にしたら０固定

    init(){
        this.bombed=false;
        this.digged=false;
        this.marked=false;
        this.checked=false;
    }

    get mutable(){//get = computed のこと statusを取得するためのメソッド
        if(this.status != 'playing') return false;//プレイ中じゃなきゃfalseで実行できない
        if(this.digged) return false;//掘ってても実行できない
        return true;
    }

    get aroundBombsNumber(){//周辺の爆弾の数を返す物　//gatherAroundCellsをGameから取得
        let bombsNumber = this.arounds.filter(element => element.bombed==true).length;//ボムがある周りのます自体をarrayに代入　そしてarrayの数を出力する
        return bombsNumber;
    }

    

    display(){//マスに表示するマーク、爆弾、周辺の爆弾の数
        if(this.marked) return '🚩';
        if(this.digged){//!外した
            if(this.bombed==true) return '💥';
            return this.aroundBombsNumber || '';
        }
        return '';
    }

    dig(){//クリックした時の処理
        if(!this.mutable) return;//undifinedが帰る
        if(this.marked) return;
        this.checked=true;
        this.$emit('edit');
        if(this.bombed==true && this.array3.length==1){
            this.bombed = false;
            this.$emit('put',this.x,this.y)//再配置　bombsの再設定
            this.dig();//else if 以下を実行し直す
        }else if(this.bombed==true && this.array3.length!=1){//ここはOK　カウント２以上で爆弾掘った時の処置
            this.digged = true;
            this.$emit('update');
        }else if(this.bombed==false && this.bombs==0){//周りに爆弾がない時の処置
            this.digged = true;
            this.$emit('open')
            this.$emit('hatch',this.x,this.y)//hatchは動いてないけど、ここまでは到達している
        }else{//周りに爆弾あるます
            this.digged = true;
            this.$emit('open')
        }
    }

    mark(){
        if(!this.mutable) return;
        this.marked = !this.marked;
    }

    
}
</script>
