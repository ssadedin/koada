<template>
  <v-app>
    <v-main>
        <v-container fluid>
            <v-row>
                <v-col>
                <v-card class="elevation-1 my-3 mx-2 codecard">
                    <v-card-title>Code Assistant</v-card-title>
                    <v-card-subtitle>Enter your instruction below, press Ctrl+J or click Generate to create a response</v-card-subtitle>
                    <v-card-text style="height: 440px">
                        <div id="editor" style="height: 370px">
                            a python function that sorts a list of integers and returns the first, third and 5th entries
                        </div>
                    </v-card-text>
                </v-card>
                </v-col>
                <v-col>
                    <v-card class="elevation-1 my-3 mx-2">
                        <v-card-title>Response</v-card-title>
                        <v-card-text>
                        <div class="responsecode" v-html="markdown_response"></div>
                        </v-card-text>
                    </v-card>
                </v-col>
            </v-row>
            <v-row>
                <v-col>
                        <v-btn @click="process()">Generate</v-btn>
                </v-col>
                <v-spacer></v-spacer>
            </v-row>
        </v-container>
    </v-main>
  </v-app>
</template>

<style>
.codecard {
    position: relative;
}
.responsecode p {
    margin: 11px 0px;
    padding: 0;
    font-size: 16px;
}

#editor { 
    position: absolute;
    top: 8em;
    right: 10px;
    left: 10px;
    height: 10em;
}
</style>

<script setup lang="ts">
import { ref } from 'vue';
import { computed, onMounted } from 'vue';

// @ts-ignore
import { llama } from '/completion.js' 
import {marked} from 'marked';

const model_response = ref('') 

const markdown_response = computed(() => {
    return marked(model_response.value)
})

onMounted(() => {
    console.log('mounted')

    // @ts-ignore
    var editor = window.ace.edit("editor");
    editor.setTheme("ace/theme/textmate");
    editor.setKeyboardHandler("ace/keyboard/vim");
    editor.session.setMode("ace/mode/markdown");
    editor.setShowPrintMargin(false);
    editor.commands.addCommand({
        name: 'generate',
        bindKey: {win: 'Ctrl-J',  mac: 'Ctrl-J'},
        exec: function(editor :any) {
            process()
        },
        readOnly: true
    });

    editor.resize()


    // @ts-ignore
    window.editor = editor;
})

async function process() {
    model_response.value = '';

    // @ts-ignore
    let inst = window.editor.getValue()

    let prompt = `
    [INST] Write code to solve the following coding problem that obeys the constraints and passes the example test cases. Please wrap your code answer using \`\`\`:
        ${inst}
    [/INST]
    `.trim()

    for await (const chunk of llama(prompt)) {
        model_response.value = model_response.value + chunk.data.content;
    }
}
</script>
