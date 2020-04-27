<template>
  <div class="ebook-reader">
      <div id="read"></div>
  </div>
</template>

<script>

import {ebookMixin} from '../../utils/mixin'
import Epub from 'epubjs'
import { getFontFamily,savefontFamily,
         getFontSize,savefontSize,
         getTheme,saveTheme } 
        from '../../utils/localStorage';
global.ePub = Epub
export default {
    mounted(){
        this.setFileName(this.$route.params.fileName.split('|').join('/'))
            .then(()=>{
                this.initEpub()
            })
    },
    mixins:[ebookMixin],
    methods:{
        initFontSize(){
            let fontSize = getFontSize(this.fileName)
            if(!fontSize){
                savefontSize(this.fileName,this.defaultFontSize)
            }else{
                this.rendition.themes.fontSize(fontSize)
                this.setDefaultFontSize(fontSize)
            }
        },
        initFontFamily(){
            let font = getFontFamily(this.fileName)
            if(!font){
                savefontFamily(this.fileName,this.defaultFontFamily)
            }else{
                this.rendition.themes.font(font)
                this.setDefaultFontFamily(font)
            }
        },
        initTheme(){
            let defaultTheme = getTheme(this.fileName)
            if(!defaultTheme){
                defaultTheme = this.themeList[0].name
                this.setDefaultTheme(defaultTheme)
                saveTheme(this.fileName,defaultTheme)
            }
            this.themeList.forEach(theme => {
                this.rendition.themes.register(theme.name,theme.style)
            })
            this.rendition.themes.select(this.defaultTheme)
        },
        initEpub(){
            const url = 'http://192.168.0.109:8081/epub/' + this.fileName + '.epub'
            this.book = new Epub(url)
            this.setCurrentBook(this.book)
            this.rendition = this.book.renderTo('read',{
                width:innerWidth,
                height:innerHeight,
                method:'default'
            })
            this.rendition.display().then(()=>{
                this.initTheme()
                this.initFontSize()
                this.initFontFamily()
            })
            this.rendition.on('touchstart',event => {
                this.touchStartX = event.changedTouches[0].clientX
                this.touchStartTime = event.timeStamp
            })
            this.rendition.on('touchend',event => {
                const offsetX = event.changedTouches[0].clientX - this.touchStartX
                const time = event.timeStamp - this.touchStartTime
                if(time < 500 && offsetX > 40){
                    this.prevPage()
                }else if(time < 500 && offsetX < -40){
                    this.nextPage()
                }else{
                    this.toggleTitleAndMenu()
                }
                event.preventDefault()
                event.stopPropagation()
            })
            this.rendition.hooks.content.register(contents => {
                Promise.all([
                    contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/daysOne.css`),
                    contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/cabin.css`),
                    contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/montserrat.css`),
                    contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/tangerine.css`)
                ]).then(()=>{})
            })
        },
        prevPage(){
            if(this.rendition){
                this.rendition.prev()
                this.hideTitleAndMenu()
            }
        },
        nextPage(){
            if(this.rendition){
                this.rendition.next()
                this.hideTitleAndMenu()
            }
        },
        toggleTitleAndMenu(){
            if(this.menuVisible){
                this.setSettingVisible(-1)
            }
            this.setMenuVisible(!this.menuVisible)
            this.setFontFamilyVisible(false)
        },
        hideTitleAndMenu(){
            this.setMenuVisible(false)
            this.setSettingVisible(-1)
        }
    }
}
</script>

<style>

 
</style>
