<template>
    <el-container style="height:calc(100% - 100px);">
        <el-main :loading="editor.loading" style="padding:0px;overflow:hidden;">
            <Editor
                v-model="editor.data"
                @init="onEditorInit"
                :lang="editor.lang.value"
                :theme="editor.theme.value"
                width="99.8%"
                height="100%"
                style="border:1px solid #f2f2f2;"
                ref="editor"></Editor>
        </el-main>
    </el-container>
</template>

<script>
import _ from 'lodash';
export default{
    name: "BitlogConsoleView",
    props: {
        model: Object
    },
    data(){
        return {
            editor: {
                data: null,
                loading: false,
                lang: {
                    value: "text",
                    list: []
                },
                theme: {
                    value: "chrome",
                    list: this.m3.EDITOR_THEME
                }
            },
        }
    },
    components:{
        Editor:require("vue2-ace-editor")
    },
    created(){
        this.init();

        /* this.eventHub.$on("WINDOW-RESIZE-EVENT",()=>{
            this.$refs.editor.editor.resize();
        });  */
    },
    methods: {
        init(){
            this.loading = true;
            let param = encodeURIComponent(JSON.stringify( this.model ));

            this.m3.callFS("/matrix/m3log/searchBitlogByTerm.js",param).then(rtn=>{
                this.editor.data = this.arrayToCsv(rtn.message.result);
                this.loading = false;
            }).catch(()=>{
                this.loading = false;
            })
        },
        arrayToCsv(data){
                    
            let lineArray = [];
            _.forEach(data, (infoArray)=> {
                let valid = (new Date(infoArray[0])).getTime() > 0;
                
                if(valid){
                    this.$set(infoArray, 0, this.moment(infoArray[0]).format('YYYY-MM-DD HH:mm:ss.SSS'));
                }

                let line = infoArray.join(", ");
                lineArray.push(line);
            });
            
            return lineArray.join("\n");
            
        },
        arrayToJson(data){
            
            this.$set(this.dt,'rows',[]);

            _.forEach(data, (v, index)=> {
                
                let valid = (new Date(v[0])).getTime() > 0;
                let time = "";

                if(valid){
                    time = this.moment(v[0]).format('YYYY-MM-DD HH:mm:ss.SSS');
                }

                this.dt.rows.push( {index: index+1, time: time, msg: v.slice(3).join(", "), file:v[1], num:v[2]} );
            });
            
        },
        markByTerm(str){
            let finalStr = str;
            let term = this.options.term.split(",");
            
            _.forEach(term,(v)=>{
                
                if(_.isEmpty(v)) return;

                let reg = new RegExp(v, 'gim');
                finalStr = finalStr.replace(reg, function(s){ return '<mark style="padding:3px;">'+s+'</mark>'; });
            })

            return finalStr;
        }
    }    
}
</script>
