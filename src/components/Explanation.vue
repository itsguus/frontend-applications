<template>
    <section class="mw" id="explanation">
        <div class="top">
            <h2>{{ title_top }}</h2>
            <p>{{ message_top }}</p>
        </div>
        <div class="bottom">
            <h2>{{ title_bottom }}</h2>
            <p>{{ message_bottom }}</p>
        </div>
    </section>
</template>


<script>
export default {
    name: "Explanation",
    data: function () {
        return {
            title_top: "Parkeerplaatsen in Nederlandse steden en dorpen",
            message_top: "Hpo",
            title_bottom: "Interactieve barchart",
            message_bottom: "bla"
        }
    }
}
</script>


<style scoped>
    .mw h2 {font-size: 2rem; font-family: "Bebas Neue"; margin-left: 0;}
    .top { margin-top: 4rem;}
    p {margin-bottom: 0.6rem; max-width: 35rem;}
    
    .bottom {text-align: right; margin: 4rem 2rem 8rem; }
    .bottom p {margin-left: auto;}

    @media only screen and (max-width: 800px) {
        .mw h2 { font-size: 1.2rem;}
    }
</style>