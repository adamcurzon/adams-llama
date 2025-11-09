<template>
    <div v-html="compiledHtml"></div>
</template>

<script>
import MarkdownIt from 'markdown-it'

export default {
    props: {
        source: {
            type: String,
            required: true
        }
    },
    data() {
        return {
            md: new MarkdownIt({
                html: true,
                linkify: true,
                typographer: true
            })
        }
    },
    computed: {
        compiledHtml() {
            return this.md.render(this.source)
        }
    }
}
</script>

<style scoped>
div {
    padding: 20px;
    background-color: rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    margin: 10px;
    text-align: left;
}
</style>
