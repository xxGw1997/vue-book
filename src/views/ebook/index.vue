<template>
    <div class="ebook">
        <ebook-title/>
        <ebook-reader />
        <ebook-Menu />
    </div>
</template>

<script>
    import EbookReader from "../../components/ebook/EbookReader"
    import EbookTitle from "../../components/ebook/EbookTitle"
    import EbookMenu from "../../components/ebook/EbookMenu"
    export default {
        components:{
            EbookTitle,
            EbookReader,
            EbookMenu
        }
    }
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
    @import "../../assets/styles/global";
 
</style>
