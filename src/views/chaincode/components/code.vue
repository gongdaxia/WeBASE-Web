/*
 * Copyright 2014-2020 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
 
<template>
    <div class="contract-code" :class="{changeActive:changeWidth }" v-loading="loading">
        <div class="contract-code-head">
            <span class="contract-code-title" v-show="codeShow" :class="{titleActive:changeWidth }">
                <span>{{contractName + '.sol'}}</span>
            </span>
            <span class="contract-code-handle" v-show="codeShow">
                <span class="contract-code-done" v-if="!contractAddress && !disabled" @click="saveCode">
                    <el-tooltip class="item" effect="dark" :content="$t('contracts.contractSaveTips')" placement="top-start">
                        <i class="wbs-icon-baocun font-16"></i>
                    </el-tooltip>
                    <span>{{this.$t("text.save")}}</span>
                </span>
                <span class="contract-code-done" @click="compile" v-if="!contractAddress && !disabled && !loading">
                    <i class="wbs-icon-bianyi font-16"></i>
                    <span>{{this.$t("text.compile")}}</span>
                </span>
                <span class="contract-code-done" @click="deploying" v-if="!contractAddress && abiFile && bin && !disabled">
                    <i class="wbs-icon-deploy font-16"></i>
                    <span>{{this.$t("text.deploy")}}</span>
                </span>
                <span class="contract-code-done" v-if="abiFile && bin && !disabled" @click="send">
                    <i class="wbs-icon-send font-16"></i>
                    <span>{{this.$t("text.send")}}</span>
                </span>
            </span>
        </div>
        <div class="contract-code-content" :class="{infoHide: !successHide}">
            <div class="contract-code-mirror" :style="{height:codeHight}" ref="codeContent">
                <div style="padding-top: 60px;text-align:center;" v-show="!codeShow">
                    <span>{{this.$t("contracts.contractOpenTips")}}</span>
                    <span>{{this.$t("contracts.createContractTips")}}</span>
                    <i class="wbs-icon-shangchuan"></i>
                    <span>{{this.$t("contracts.uploadContractTips")}}</span>
                    <strong><i class="wbs-icon-Addfile"></i></strong>
                </div>
                <div class="ace-editor" ref="ace" v-show="codeShow"></div>
            </div>
            <div class="contract-info" v-show="successHide" :style="{height:infoHeight + 'px'}">
                <div class="move" @mousedown="dragDetailWeight($event)" @mouseup="resizeCode"></div>
                <div class="contract-info-title" @mouseover="mouseHover=!mouseHover" @mouseout="mouseHover=!mouseHover" v-show="abiFile||contractAddress">
                    <i :class="[showCompileText?'el-icon-caret-bottom':'el-icon-caret-top']" @click="collapse">

                    </i>
                </div>
                <div>
                    <div class="contract-info-list1" v-html="compileinfo">
                    </div>
                    <div class="contract-info-list1" style="color: #f00" v-show="errorInfo">
                        {{errorInfo}}
                    </div>
                    <div class="contract-info-list1" style="color: #f00" v-show="errorInfo">
                        <span style="display:inline-block;width:calc(100% - 120px);word-wrap:break-word" v-for="(item, index) in errorMessage">{{index+1}}、{{item}}</span>
                    </div>
                    <div style="color: #68E600;padding-bottom: 15px;" v-show="abiFileShow">{{successInfo}}</div>
                    <div class="contract-info-list" v-show="contractAddress">
                        <span class="contract-info-list-title" style="color: #0B8AEE">contractAddress
                            <i class="wbs-icon-copy font-12 copy-public-key" @click="copyKey(contractAddress)" :title="$t('text.copy')"></i>
                        </span>
                        <span style="display:inline-block;width:calc(100% - 120px);word-wrap:break-word">{{contractAddress}}</span>
                    </div>
                    <div class="contract-info-list" v-show="abiFile">
                        <span class="contract-info-list-title" style="color: #0B8AEE">contractName
                            <i class="wbs-icon-copy font-12 copy-public-key" @click="copyKey(contractName)" :title="$t('text.copy')"></i> </span>
                        <span style="display:inline-block;width:calc(100% - 120px);word-wrap:break-word">{{contractName}}</span>
                    </div>
                    <div class="contract-info-list" v-show="abiFile">
                        <span class="contract-info-list-title" style="color: #0B8AEE">abi
                            <i class="wbs-icon-copy font-12 copy-public-key" @click="copyKey(abiFile)" :title="$t('text.copy')"></i>
                        </span>
                        <span class="showText" ref="showAbiText">
                            {{abiFile}}
                        </span>
                        <i :class="[ showAbi ? 'el-icon-arrow-down': 'el-icon-arrow-up','font-13','cursor-pointer','visibility-wrapper'] " v-if="complieAbiTextHeight" @click="showAbiText"></i>
                    </div>
                    <div class="contract-info-list" style="border-bottom: 1px solid #e8e8e8" v-show="abiFile">
                        <span class="contract-info-list-title" style="color: #0B8AEE">bytecodeBin
                            <i class="wbs-icon-copy font-12 copy-public-key" @click="copyKey(bytecodeBin)" :title="$t('text.copy')"></i>
                        </span>
                        <span class="showText" ref="showBinText">
                            {{bytecodeBin}}
                        </span>
                        <i :class="[ showBin ? 'el-icon-arrow-down': 'el-icon-arrow-up','font-13','cursor-pointer','visibility-wrapper'] " v-if="complieBinTextHeight" @click="showBinText"></i>
                    </div>
                </div>
            </div>
        </div>
        <el-dialog v-dialogDrag :title="$t('contracts.sendTransaction')" :visible.sync="dialogVisible" width="580px" :before-close="sendClose" v-if="dialogVisible" center class="send-dialog">
            <v-transaction @success="sendSuccess($event)" @close="handleClose" ref="send" :data="data" :abi='abiFile' :version='version' :address='uploadAddress'></v-transaction>
        </el-dialog>
        <el-dialog v-dialogDrag :title="$t('contracts.changeUser')" :visible.sync="dialogUser" width="350px" v-if="dialogUser" center class="send-dialog">
            <v-user @change="deployContract($event)" @close="userClose" :abi='abiFile'></v-user>
        </el-dialog>
        <v-editor v-if='editorShow' :show='editorShow' :data='editorData' :input='editorInput' :editorOutput="editorOutput" :sendConstant="sendConstant" @close='editorClose'></v-editor>
        <v-upload v-if='uploadFileAdrShow' :show='uploadFileAdrShow' @close='uploadClose' @success='uploadSuccess($event)'></v-upload>
    </div>
</template>

<script>
import ace from "ace-builds";
import "ace-builds/webpack-resolver";
import "ace-builds/src-noconflict/theme-chrome";
import "ace-builds/src-noconflict/mode-javascript";
import "ace-builds/src-noconflict/ext-language_tools";
require("ace-mode-solidity/build/remix-ide/mode-solidity");
let Mode = require("ace-mode-solidity/build/remix-ide/mode-solidity").Mode;
import errcode from "@/util/errcode";
let Base64 = require("js-base64").Base64;
import constant from "@/util/constant";
import editor from "@/components/editor"
import uploadFileAdr from "../dialog/uploadFileAdr"
import Bus from '@/bus'
import web3 from "@/util/ethAbi"
import router from "@/router"
import {
    addChaincode,
    getDeployStatus,
    setCompile,
    editChain,
    addFunctionAbi,
    backgroundCompile
} from "@/util/api";
import transaction from "@/components/sendTransaction";
import changeUser from "../dialog/changeUser";

export default {
    name: "codes",
    props: ["show", "changeStyle"],
    components: {
        "v-transaction": transaction,
        "v-user": changeUser,
        "v-editor": editor,
        "v-upload": uploadFileAdr
    },
    data: function () {
        return {
            successHide: true,
            loading: false,
            content: "",
            successShow: true,
            errorShow: false,
            dialogVisible: false,
            code: "",
            status: 0,
            abiFile: "",
            bin: "",
            contractAddress: "",
            contractName: "",
            infoHeight: 250,
            contractList: [],
            dialogUser: false,
            compileinfo: "",
            errorInfo: "",
            errorMessage: "",
            successInfo: "",
            abiFileShow: false,
            bytecodeBin: "",
            aceEditor: null,
            themePath: "ace/theme/chrome",
            modePath: "ace/mode/solidity",
            data: null,
            codeShow: false,
            version: "",
            saveShow: false,
            editorShow: false,
            editorData: null,
            editorInput: null,
            editorOutput: null,
            uploadFileAdrShow: false,
            uploadAddress: "",
            disabled: false,
            showAbi: true,
            showBin: true,
            complieAbiTextHeight: false,
            complieBinTextHeight: false,
            mouseHover: false,
            showCompileText: true,
            sendConstant: null
        };
    },
    beforeDestroy: function () {
        Bus.$off("select")
        Bus.$off("noData")
    },
    beforeMount() {

    },
    mounted: function () {
        if (localStorage.getItem("root") === "admin" || localStorage.getItem("root") === "developer") {
            this.disabled = false
        } else {
            this.disabled = true
        }
        this.initEditor();
        Bus.$on('select', data => {
            this.codeShow = true;
            this.refreshMessage();
            this.code = "";
            this.version = "";
            this.status = null;
            this.abiFile = "";
            this.contractAddress = "";
            this.errorMessage = "";
            this.contractName = "";
            this.content = "";
            this.bin = "";
            this.data = data;
            this.code = Base64.decode(data.contractSource);
            this.content = this.code;
            this.aceEditor.setValue(this.content);
            this.status = data.contractStatus;
            this.abiFile = data.contractAbi;
            this.contractAddress = data.contractAddress;
            this.errorMessage = data.description || "";
            this.contractName = data.contractName;
            this.bin = data.contractBin;
            this.bytecodeBin = data.bytecodeBin || "";
            this.version = data.contractVersion;
            this.complieAbiTextHeight = false;
            this.complieBinTextHeight = false;
            // this.$refs['showAbiText'].style.overflow = 'hidden'
            // this.$refs['showBinText'].style.overflow = 'hidden'
            // if (data.contractAbi) {
            //     this.$nextTick(() => {
            //         if (this.$refs['showAbiText'].offsetHeight >= 72) {
            //             this.complieAbiTextHeight = true
            //         }
            //         if (this.$refs['showBinText'].offsetHeight >= 72) {
            //             this.complieBinTextHeight = true
            //         }

            //     })
            // }

        })
        Bus.$on("noData", data => {
            this.codeShow = false;
            this.refreshMessage();
            this.code = "";
            this.version = "";
            this.status = null;
            this.abiFile = "";
            this.contractAddress = "";
            this.errorMessage = "";
            this.contractName = "";
            this.content = "";
            this.bin = "";
        })
    },
    watch: {
        content: function (val) {
            let data = Base64.decode(this.data.contractSource);
            if (data != val) {
                this.saveShow = true
            } else {
                this.saveShow = false
            }
        },
        successHide: function (val) {
            if (val) {
                this.infoHeight = 250;
            } else {
                this.infoHeight = 0;
            }
        }
    },
    computed: {
        codeHight: function () {
            if (this.infoHeight) {
                return `calc(100% - ${this.infoHeight}px)`;
            } else {
                return `100%`;
            }
        },
        changeWidth() {
            if (this.changeStyle) {
                return this.changeStyle;
            } else {
                return false;
            }
        },
        tipShow() {
            return !this.show;
        }
    },
    methods: {
        initEditor: function () {
            let _this = this
            this.aceEditor = ace.edit(this.$refs.ace, {
                fontSize: 14,
                fontFamily: "Consolas,Monaco,monospace",

                theme: this.themePath,
                mode: this.modePath,
                tabSize: 4,
                useSoftTabs: true
            });
            this.aceEditor.setOptions({
                enableSnippets: true,
                enableLiveAutocompletion: true,
                enableBasicAutocompletion: true,
                autoScrollEditorIntoView: true,
                copyWithEmptySelection: true
            });
            this.aceEditor.commands.addCommand({
                name: 'save',
                bindKey: { win: 'Ctrl-S', mac: 'Command-S' },
                exec: function (editor) {
                    if (_this.data.contractStatus != 2) {
                        _this.saveCode()
                    }
                },
            });
            this.aceEditor.commands.addCommand({
                name: 'compile',
                bindKey: { win: "Alt-C", mac: "Option-C" },
                exec: function (editor) {
                    if (!_this.contractAddress && !_this.disabled) {
                        _this.compile();
                    }
                }
            });
            this.aceEditor.commands.addCommand({
                name: 'deploying',
                bindKey: { win: "Alt-D", mac: "Option-D" },
                exec: function (editor) {
                    if (!_this.contractAddress && _this.abiFile && _this.bin && !_this.disabled) {
                        _this.deploying();
                    }
                }
            });
            this.aceEditor.commands.addCommand({
                name: 'send',
                bindKey: { win: "Alt-T", mac: "Option-T" },
                exec: function (editor) {
                    if (_this.abiFile && _this.bin && !_this.disabled) {
                        _this.send();
                    }
                }
            });
            let editor = this.aceEditor.alignCursors();
            this.aceEditor.getSession().setUseWrapMode(true);
            this.aceEditor.getSession().on("change", this.changeAce);
            this.aceEditor.on("blur", this.blurAce);
            this.aceEditor.resize();
        },
        blurAce: function () {
            let data = Base64.encode(this.content);
            if (this.data.contractSource != data && this.data.contractStatus != 2) {
                this.saveCode()
            }
        },
        saveCode: function () {
            this.data.contractSource = Base64.encode(this.content);
            Bus.$emit("save", this.data)
        },
        resizeCode: function () {
            this.aceEditor.setOptions({
                maxLines:
                    Math.ceil(this.$refs.codeContent.offsetHeight / 17) - 1
            });
            this.aceEditor.resize();
        },
        dragDetailWeight: function (e) {
            let startY = e.clientY,
                infoHeight = this.infoHeight;
            document.onmousemove = e => {
                let moveY = startY - e.clientY;
                this.infoHeight = infoHeight + moveY;
            };
            document.onmouseup = e => {
                document.onmousemove = null;
                document.onmouseup = null;
            };
            this.aceEditor.setOptions({
                maxLines:
                    Math.ceil(this.$refs.codeContent.offsetHeight / 17) - 1,
                minLines: 9
            });
        },
        upLoadAdr: function () {
            this.uploadFileAdrShow = true
        },
        uploadClose: function () {
            this.uploadFileAdrShow = false
        },
        uploadSuccess: function (val) {
            this.dialogVisible = true;
            this.uploadAddress = val
        },
        sendSuccess: function (val) {
            this.sendConstant = val.constant;
            this.uploadAddress = "";
            this.dialogVisible = false;
            this.editorShow = true;
            this.editorData = null;
            this.editorData = val.resData;
            this.editorInput = val.input;
            this.editorOutput = val.data.outputs;
            if (val && val.contractAddress) {
                this.contractAddress = val.contractAddress;
                this.data.contractAddress = val.contractAddress;
                Bus.$emit("send", this.data)
            }
        },
        editorClose: function () {
            this.editorShow = false;
        },
        changeAce: function () {
            this.content = this.aceEditor.getSession().getValue();
        },

        findImports: function (path) {
            this.contractList = this.$store.state.contractDataList
            let arry = path.split("/");
            let newpath = arry[arry.length - 1];
            let num = 0;
            if (arry.length > 1) {
                let newPath = arry[0]
                let oldPath = arry[arry.length - 1]
                let importArry = []
                this.contractList.forEach(value => {
                    if (value.contractPath == newPath) {
                        importArry.push(value)
                    }
                })
                if (importArry.length) {
                    for (let i = 0; i < importArry.length; i++) {
                        if (oldPath == importArry[i].contractName + ".sol") {
                            return {
                                contents: Base64.decode(
                                    importArry[i].contractSource
                                )
                            };
                        }
                    }
                } else {
                    return { error: "File not found" };
                }
            } else {
                let newpath = arry[arry.length - 1];
                let newArry = []
                this.contractList.forEach(value => {
                    if (value.contractPath == this.data.contractPath) {
                        newArry.push(value)
                    }
                })
                if (newArry.length > 1) {
                    for (let i = 0; i < newArry.length; i++) {
                        if (newpath == newArry[i].contractName + ".sol") {
                            return {
                                contents: Base64.decode(
                                    newArry[i].contractSource
                                )
                            };
                        }
                    }
                    for (let i = 0; i < this.contractList.length; i++) {
                        if (newpath == this.contractList[i].contractName + ".sol") {
                            return {
                                contents: Base64.decode(this.contractList[i].contractSource)
                            };
                        } else {
                            num++;
                        }
                    }
                    if (num) {
                        return { error: "File not found" };
                    }
                } else {
                    let num1 = 0
                    for (let i = 0; i < this.contractList.length; i++) {
                        if (newpath == this.contractList[i].contractName + ".sol") {
                            return {
                                contents: Base64.decode(this.contractList[i].contractSource)
                            };
                        } else {
                            num1++
                        }
                    }
                    if (num1) {
                        return { error: "File not found" };
                    }
                }
            }
        },
        compile() {
            this.loading = true;
            let version = this.$store.state.versionData;
            if (version && version.net !== 0) {
                this.compileHighVersion()
            } else {
                setTimeout(() => {
                    this.compileLowVersion()
                }, 500)

            }
        },
        compileHighVersion() {
            let that = this

            this.refreshMessage();
            this.contractList = this.$store.state.contractDataList
            let content = "";
            let output;
            let input = {
                language: "Solidity",
                settings: {
                    outputSelection: {
                        "*": {
                            "*": ["*"]
                        }
                    }
                }
            };
            input.sources = {};
            input.sources[this.contractName + ".sol"] = {};
            let libs = [];
            input.sources[this.contractName + ".sol"] = {
                content: this.content
            };
            let w = this.$store.state.worker;
            w.postMessage({
                cmd: "compile",
                input: JSON.stringify(input),
                list: this.$store.state.contractDataList,
                path: this.data.contractPath
            });
            w.addEventListener('message', function (ev) {
                if (ev.data.cmd == 'compiled') {
                    that.loading = false
                    output = JSON.parse(ev.data.data);
                    if (output && output.contracts && JSON.stringify(output.contracts) != "{}") {
                        that.status = 1;
                        if (output.contracts[that.contractName + ".sol"]) {
                            that.changeOutput(
                                output.contracts[that.contractName + ".sol"]
                            );
                        }
                    } else {
                        that.errorMessage = output.errors;
                        that.errorInfo = that.$t("contracts.contractCompileFail");
                        that.loading = false;
                    }
                } else {
                    console.log(ev.data);
                    console.log(JSON.parse(ev.data.data))
                }
            });
            w.addEventListener("error", function (ev) {
                that.errorInfo = ev;
                that.errorMessage = ev;
                that.compileShow = true;
                that.loading = false;
            })
        },
        compileLowVersion: function () {
            // this.loading = true;
            let wrapper = require("solc/wrapper");
            let solc = wrapper(window.Module);
            this.refreshMessage();
            for (let i = 0; i < constant.COMPILE_INFO.length; i++) {
                this.compileinfo = this.compileinfo + constant.COMPILE_INFO[i];
            }
            this.contractList = this.$store.state.contractDataList
            let content = "";
            let output;
            let input = {
                language: "Solidity",
                settings: {
                    outputSelection: {
                        "*": {
                            "*": ["*"]
                        }
                    }
                }
            };
            input.sources = {};
            input.sources[this.contractName + ".sol"] = {};
            let libs = [];
            input.sources[this.contractName + ".sol"] = {
                content: this.content
            };
            try {
                output = JSON.parse(solc.compile(JSON.stringify(input), { import: this.findImports }));
            } catch (error) {
                this.errorInfo = error;
                this.errorMessage = error;
                this.compileShow = true;
                this.loading = false;
            }
            setTimeout(() => {
                if (output && JSON.stringify(output.contracts) != "{}") {
                    this.status = 1;
                    if (output.contracts[this.contractName + ".sol"]) {
                        this.changeOutput(
                            output.contracts[this.contractName + ".sol"]
                        );
                    }
                } else {
                    this.errorMessage = output.errors;
                    this.errorInfo = this.$t("contracts.contractCompileFail");
                    this.loading = false;
                }
            }, 500)
        },
        changeOutput: function (obj) {
            if (JSON.stringify(obj) !== "{}") {
                if (obj.hasOwnProperty(this.contractName)) {
                    let compiledMap = obj[this.contractName]
                    this.abiFileShow = true;
                    this.successInfo = this.$t("contracts.compileSuccess");
                    this.abiFile = compiledMap.abi;
                    this.abiFile = JSON.stringify(this.abiFile);
                    this.bin = compiledMap.evm.deployedBytecode.object;
                    this.bytecodeBin = compiledMap.evm.bytecode.object;
                    this.data.contractAbi = this.abiFile;
                    this.data.contractBin = this.bin;
                    this.data.contractSource = Base64.encode(this.content);
                    this.$set(this.data, "bytecodeBin", this.bytecodeBin);
                    this.loading = false;
                    Bus.$emit("compile", this.data);
                    this.setMethod();
                } else {
                    this.$message({
                        type: "error",
                        message: this.$t("contracts.conmileNameError")
                    })
                    this.errorInfo = this.$t("contracts.contractCompileFail");
                    this.compileShow = true;
                    this.loading = false;
                }
            } else {
                this.errorInfo = this.$t("contracts.contractCompileFail");
                this.compileShow = true;
                this.loading = false;
            }
        },
        refreshMessage: function () {
            this.abiFileShow = false;
            this.errorInfo = "";
            this.compileinfo = "";
            this.abiFile = "";
            this.contractAddress = "";
        },
        deploying: function () {
            this.dialogUser = true;
        },
        userClose: function () {
            this.dialogUser = false;
        },
        setMethod: function () {
            let Web3EthAbi = web3;
            let arry = [];
            if (this.abiFile) {
                let list = JSON.parse(this.abiFile);
                list.forEach(value => {
                    if (value.name && value.type == 'function') {
                        let data = {}
                        let methodId;
                        if (localStorage.getItem("encryptionId") == 1) {
                            methodId = Web3EthAbi.smEncodeFunctionSignature({
                                name: value.name,
                                type: value.type,
                                inputs: value.inputs
                            });
                        } else {
                            methodId = Web3EthAbi.encodeFunctionSignature({
                                name: value.name,
                                type: value.type,
                                inputs: value.inputs
                            });
                        }
                        data.methodId = methodId;
                        data.abiInfo = JSON.stringify(value);
                        data.methodType = value.type
                        arry.push(data)
                    } else if (value.name && value.type == 'event') {
                        let data = {}
                        let methodId;
                        if (localStorage.getItem("encryptionId") == 1) {
                            methodId = Web3EthAbi.smEncodeEventSignature({
                                name: value.name,
                                type: value.type,
                                inputs: value.inputs
                            });
                        } else {
                            methodId = Web3EthAbi.encodeEventSignature({
                                name: value.name,
                                type: value.type,
                                inputs: value.inputs
                            });
                        }
                        data.methodId = methodId;
                        data.abiInfo = JSON.stringify(value);
                        data.methodType = value.type
                        arry.push(data)
                    }
                })
                if (arry.length) {
                    this.addAbiMethod(arry)
                }
            }
        },
        addAbiMethod: function (list) {
            let data = {
                groupId: localStorage.getItem("groupId"),
                methodList: list
            }
            addFunctionAbi(data).then(res => {
                if (res.data.code === 0) {
                    console.log("method 保存成功！")
                } else {
                    this.$message({
                        message: this.$chooseLang(res.data.code),
                        type: "error",
                        duration: 2000
                    });
                }
            }).catch(err => {
                this.$message({
                    message: this.$t('text.systemError'),
                    type: "error",
                    duration: 2000
                });
            })
        },
        deployContract(val) {
            if (val && !val.userId) {
                this.$message({
                    type: "info",
                    message: this.$t('contracts.addPrivateKeyInfo')
                });
                router.push("/privateKeyManagement");
                return
            }
            this.loading = true;
            let reqData = {
                groupId: localStorage.getItem("groupId"),
                contractBin: this.bin,
                bytecodeBin: this.bytecodeBin,
                contractAbi: this.abiFile,
                contractSource: Base64.encode(this.content),
                user: val.userId,
                contractName: this.contractName,
                contractId: this.data.contractId,
                contractPath: this.data.contractPath,
                account: localStorage.getItem("user")
            };
            this.version = val.version
            if (val.params.length) {
                reqData.constructorParams = val.params;
            }
            getDeployStatus(reqData)
                .then(res => {
                    this.loading = false;
                    if (res.data.code === 0) {
                        this.abiFileShow = true;
                        this.status = 2;
                        this.contractAddress = res.data.data.contractAddress
                        this.successInfo = this.$t("contracts.deploySuccess");
                        this.abiFile = res.data.data.contractAbi;
                        this.bin = res.data.data.contractBin;
                        this.$message({
                            message: this.$t("contracts.contractDeploySuccess"),
                            type: "success"
                        });
                        this.data.contractAbi = this.abiFile;
                        this.data.contractBin = this.bin;
                        this.data.contractSource = Base64.encode(this.content);
                        this.data.contractAddress = this.contractAddress;
                        this.data.contractVersion = this.version;
                        Bus.$emit("deploy", this.data)
                    } else {
                        this.status = 3;
                        this.$message({
                            message: this.$chooseLang(res.data.code),
                            type: "error",
                            duration: 2000
                        });
                    }
                })
                .catch(err => {
                    this.status = 3;
                    this.loading = false;
                    this.$message({
                        message: this.$t('text.systemError'),
                        type: "error",
                        duration: 2000
                    });
                });
        },
        foldInfo: function (val) {
            this.successHide = val;
        },
        send: function () {
            this.dialogVisible = true;
        },
        handleClose: function () {
            this.dialogVisible = false;
        },
        sendClose: function () {
            this.$refs.send.close();
        },
        showAbiText() {
            this.showAbi = !this.showAbi
            if (this.$refs['showAbiText'].style.overflow === 'visible') {
                this.$refs['showAbiText'].style.overflow = 'hidden'
            } else if (this.$refs['showAbiText'].style.overflow === '' || this.$refs['showAbiText'].style.overflow === 'hidden') {
                this.$refs['showAbiText'].style.overflow = 'visible'
            }
        },
        showBinText() {
            this.showBin = !this.showBin
            if (this.$refs['showBinText'].style.overflow === 'visible') {
                this.$refs['showBinText'].style.overflow = 'hidden'
            } else if (this.$refs['showBinText'].style.overflow === '' || this.$refs['showBinText'].style.overflow === 'hidden') {
                this.$refs['showBinText'].style.overflow = 'visible'
            }
        },
        showHover() {

        },
        hiddenHover() {

        },
        collapse() {
            this.showCompileText = !this.showCompileText
            if (this.showCompileText) {
                this.infoHeight = 250

            } else if (!this.showCompileText) {
                this.infoHeight = 50
            }
            this.$nextTick(() => {
                this.resizeCode()
            })

        },
        copyKey(val) {
            if (!val) {
                this.$message({
                    type: "fail",
                    showClose: true,
                    message: this.$t("text.copyErrorMsg"),
                    duration: 2000
                });
            } else {
                this.$copyText(val).then(e => {
                    this.$message({
                        type: "success",
                        showClose: true,
                        message: this.$t("text.copySuccessMsg"),
                        duration: 2000
                    });
                });
            }
        }

    }
};
</script>
<style scoped>
.contract-code {
    height: 100%;
    border-left: 1px solid #eee;
    box-sizing: border-box;
}
.changeActive {
    padding-left: 0 !important;
}
.contract-code-head {
    width: 100%;
    height: 48px;
    line-height: 48px;
    border-bottom: 2px solid #e7ebf0;
    background-color: #fff;
    overflow: hidden;
}
.contract-code-done {
    display: inline-block;
    margin-right: 10px;
    cursor: pointer;
}
.contract-code-done i {
    vertical-align: middle;
}
.contract-code-done span {
    font-size: 12px;
    color: #9da2ab;
    vertical-align: middle;
}
.contract-no-content {
    border-left: 1px solid #e7ebf0;
    height: calc(100% - 50px);
    box-sizing: border-box;
}
.contract-code-content {
    border-left: 1px solid #e7ebf0;
    height: calc(100% - 50px);
    box-sizing: border-box;
}
.contract-code-mirror {
    width: 100%;
    height: 70%;
}
.contract-info {
    position: relative;
    padding-top: 20px;
    text-align: left;
    border-top: 1px solid #ddd;
    box-sizing: border-box;
    overflow: auto;
}
.contract-info-content {
    height: 100%;
    overflow-y: scroll;
    box-sizing: border-box;
}
.contract-code-title {
    float: left;
    font-weight: bold;
    font-size: 18px;
    color: #36393d;
    padding-left: 20px;
}
.contract-code-handle {
    float: right;
    padding-right: 20px;
}
.contract-info-title {
    text-align: center;
}
.move {
    position: absolute;
    width: 100%;
    height: 3px;
    top: 0;
    left: 0;
    z-index: 9999;
    cursor: s-resize;
}
.contract-info-title i {
    padding-left: 8px;
    font-size: 10px;
    color: #aeb1b5;
    cursor: pointer;
}
.contract-info-title span {
    font-size: 16px;
    font-weight: bold;
    color: #36393d;
}
.contract-info-list {
    padding: 5px 20px;
    width: 90%;
    margin: 0 auto;
    border: 1px solid #e8e8e8;
    border-bottom: none;
    position: relative;
}
.contract-info-list-title {
    display: inline-block;
    width: 105px;
    vertical-align: top;
}
.contract-info-list-title::after {
    display: block;
    content: "";
    clear: both;
}
.ace-editor {
    height: 100% !important;
    position: relative;
    text-align: left;
    letter-spacing: 0.1px;
    text-rendering: geometricPrecision;
    font-feature-settings: "liga" 0;
    font-variant-ligatures: none;
    font: 14px / normal "Monaco", "Menlo", "Ubuntu Mono", "Consolas",
        "source-code-pro", monospace !important;
}
.ace-editor >>> .ace_print-margin {
    display: none;
    text-rendering: geometricPrecision;
}
.infoHide {
    height: calc(100% - 50px);
}
.code-spread {
    position: absolute;
    width: 33px;
    height: 33px;
    line-height: 33px;
    left: 412px;
    bottom: 0;
    border: 1px solid #e8e8e8;
    color: #aeb1b5;
    background-color: #fff;
    text-align: center;
    z-index: 9999;
    cursor: pointer;
}
.code-spread i {
    font-size: 12px;
}
.contract-info {
    background-color: #fff;
}
.titleActive {
    padding-left: 40px;
}
.send-dialog >>> .el-dialog--center .el-dialog__body {
    padding: 5px 25px 20px;
}
.showText {
    display: inline-block;
    width: calc(100% - 120px);
    word-wrap: break-word;
    max-height: 73px;
    overflow: hidden;
}
.visibility-wrapper {
    position: absolute;
    bottom: 10px;
}
.copy-public-key {
    float: right;
}
.contract-font {
    color: #777;
    cursor: pointer;
}
.contract-font:hover {
    color: #111;
}
.contract-font:active {
    color: #111;
}
.contract-font:visited {
    color: #111;
}
</style>
