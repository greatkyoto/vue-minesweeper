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
    <div :class="{digged}" @click="dig()" @contextmenu.prevent="mark()"> <!--v-onの省略記法-->
        {{display()}}
    </div>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";
import {Status} from './Game.vue';//vueの導入

@Component({props: ['status', 'arounds', 'x', 'y']})//クラスがvueコンポーネントであることを示す propsはプロパティを受け取れるようにしたもの。GameからCellに
export default class Cell extends Vue{
    readonly status!:Status;
    readonly arounds!: Cell[];
    readonly x!: number;
    readonly y!: number;

    bomb: boolean = false;
    digged: boolean = false;
    marked: boolean = false;

    get mutable(){//惚れないようにマークすること？？
        if(this.status != 'playing') return false;//プレイ中じゃなきゃfalseで実行できない
        if(this.digged) return false;//掘ってても実行できない
        return true;
    }

    get aroundBombsNumber(){//周辺の爆弾の数は初期値０

        return 0;
    }

    display(){//マスに表示するマーク、爆弾、周辺の爆弾の数
        if(this.marked) return '🚩';
        if(!this.digged){
            if(this.bomb) return '💥';
            return this.aroundBombsNumber || '';//||は何を指すのか
        }

        return '';
    }


    dig(){
        if(!this.mutable) return;//undifinedが帰る
        if(this.marked) return;
        
        this.digged = true;
        this.$emit('update'); //CellからGameに
    }

    mark(){
        if(!this.mutable) return;
        this.marked = !this.marked;
    }
}
</script>
