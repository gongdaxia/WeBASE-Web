<template>
    <div>
        <el-form :model="contractEventForm" :rules="rules" ref="contractEventForm" label-width="135px" class="demo-ruleForm">
            <el-form-item :label="$t('table.contractAddress')" prop="contractAddress">
                <el-input v-model.trim="contractEventForm.contractAddress" style="width: 240px;" clearable disabled></el-input>
                <el-tooltip class="item" effect="dark" :content="$t('contracts.contractAddress')" placement="right">
                    <i class="el-icon-info"></i>
                </el-tooltip>
            </el-form-item>
            <el-form-item :label="$t('table.fromBlock')" prop="fromBlock">
                <el-input-number v-model.trim="contractEventForm.fromBlock" style="width: 240px;" clearable controls-position="right" :min="1"></el-input-number>
                <!-- <el-tooltip class="item" effect="dark" :content="$t('contracts.fromBlock')" placement="right">
                    <i class="el-icon-info"></i>
                </el-tooltip> -->
            </el-form-item>
            <el-form-item :label="$t('table.toBlock')" prop="toBlock">
                <el-input-number v-model.trim="contractEventForm.toBlock" style="width: 240px;" clearable controls-position="right" :min="1"></el-input-number>
            </el-form-item>
            <el-form-item :label="$t('table.eventName')" prop="eventName">
                <el-select v-model="contractEventForm.eventName" :placeholder="$t('placeholder.selected')" style="width: 240px;" @change="changeEventName">
                    <el-option v-for="item in eventNameList" :key="item.value" :label="item.label" :value="item.value">
                    </el-option>
                </el-select>
                <el-tooltip class="item" effect="dark" :content="$t('contracts.eventParam')" placement="right">
                    <i class="el-icon-info"></i>
                </el-tooltip>
                <li v-for="item in inputList" class="event-info">
                    <div v-if="item.indexed">
                        <div>{{item.name}}:</div>
                        <el-input v-model="item.value" :placeholder="item.type" style="width: 240px;" @input="e => (item.msgObj = isType(e,item.type))"></el-input>
                        <span v-if="item.msgObj&&!item.msgObj.is" class="font-color-ed5454 font-12" style="display:inline-block">
                            {{item.msgObj.msg}}
                        </span>
                    </div>
                </li>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="modelClose">{{$t('text.cancel')}}</el-button>
            <el-button type="primary" @click="submit('contractEventForm')" :loading="loading">{{$t('text.confirm')}}</el-button>
        </div>
    </div>
</template>

<script>
import { checkEvent } from "@/util/api";
import { validateEvent } from "@/util/validate";
export default {
    name: 'contractEventDialog',

    components: {
    },

    props: ['contractInfo'],

    data() {
        return {
            loading: false,
            contractEventForm: {
                groupId: '',
                contractAbi: this.contractInfo.contractAbi,
                contractAddress: this.contractInfo.contractAddress,
                fromBlock: '',
                toBlock: '',
                eventName: ''
            },
            groupList: localStorage.getItem("cluster")
                ? JSON.parse(localStorage.getItem("cluster"))
                : [],
            groupId: localStorage.getItem("groupId"),
            inputList: []
        }
    },

    computed: {
        rules() {
            var obj = {
                // contractAbi: [
                //     {
                //         required: true,
                //         message: this.$t('rule.contractAbi'),
                //         trigger: "blur"
                //     }
                // ],
                // contractAddress: [
                //     {
                //         required: true,
                //         message: this.$t('rule.contractAddress'),
                //         trigger: "blur"
                //     },
                // ],
                // fromBlock: [
                //     {
                //         required: true,
                //         message: this.$t('rule.fromBlock'),
                //         trigger: "blur"
                //     },
                // ],
                // toBlock: [
                //     {
                //         required: true,
                //         message: this.$t('rule.toBlock'),
                //         trigger: "blur"
                //     },
                // ],

                eventName: [
                    {
                        required: true,
                        message: this.$t('rule.eventName'),
                        trigger: "change"
                    },
                ]
            }
            return obj
        },
        eventNameList() {
            var abiList = JSON.parse(this.contractInfo.contractAbi)
            var options = []
            abiList.forEach(item => {
                if (item.type == 'event') {
                    var param = [];
                    item.inputs.forEach(it => {
                        param.push(`${it.type}`)

                    })
                    options.push({
                        label: `${item.name}`,
                        value: `${item.name}(${param.join(',')})`
                    })
                }
            });
            return options
        }
    },

    watch: {
    },

    created() {
    },

    mounted() {

    },

    methods: {
        changeEventName($event) {
            var eventName = $event.replace(/\((.+?)\)/g, '')
            var abiList = JSON.parse(this.contractInfo.contractAbi)
            var eventListInfo = []
            abiList.forEach(item => {
                if (item.type === "event") {
                    eventListInfo.push(item)
                }
            })
            var inputList = []
            eventListInfo.forEach(item => {
                if (item.name === eventName) {
                    inputList = item.inputs
                }
            })
            inputList.forEach(item => {
                if (item.indexed) {
                    item.value = null;
                }
            })
            this.inputList = inputList;
        },
        submit(formName) {
            this.$refs[formName].validate(valid => {
                if (valid) {
                    this.queryAdd()
                } else {
                    return false;
                }
            })
        },
        queryAdd() {
            try {
                JSON.parse(this.contractEventForm.contractAbi)
            } catch (error) {
                this.$message({
                    type: 'error',
                    message: error
                })
                return
            }
            var topicObj = {
                eventName: this.contractEventForm.eventName
            }
            var indexedList = []
            this.inputList.forEach((item, index) => {
                if (item.indexed) {
                    indexedList.push(item)

                }
            })
            indexedList.forEach((item, index) => {
                topicObj[`indexed${index+1}`] = {
                    type: item.type,
                    value: item.value
                }
            })
            this.loading = true;
            let param = Object.assign({}, this.contractEventForm, {
                contractAbi: JSON.parse(this.contractEventForm.contractAbi),
                topics: topicObj,
                groupId: this.groupId
            })
            delete param.eventName;
            checkEvent(param)
                .then(res => {
                    this.loading = false;
                    if (res.data.code === 0) {
                        this.modelClose()
                        this.$emit("checkEventSuccess", res.data.data)
                    } else {
                        this.$message({
                            type: "error",
                            message: this.$chooseLang(res.data.code)
                        });
                    }
                })
                .catch(err => {
                    this.$message({
                        type: "error",
                        message: this.$t('text.systemError')
                    });
                });
        },
        modelClose() {
            this.$emit('checkEventClose')
        },
        isType(val, type) {
            return validateEvent(type, val)
        },
        isnumber(val) {
            // console.log(val, type)
            // var uintReg = RegExp(/uint/);
            // var bytesReg = RegExp(/bytes/);
            // var boolReg = RegExp(/bool/);
            // if(uintReg.test(type) || bytesReg.test(type)){
            //     val = val.replace(/[^0-9]/gi, "");
            //     return val;
            // }

            return val

        },
        transform(type, val) {
            if (!val) {
                return val = null;
            }
            var uintReg = RegExp(/uint/);
            var bytesReg = RegExp(/bytes/);
            var boolReg = RegExp(/bool/);
            if (uintReg.test(type) || bytesReg.test(type)) {
                try {
                    return val = +val
                } catch (error) {
                    console.log(console.error())
                }

            } else if (boolReg.test(type)) {
                return val = eval(val.toLowerCase())
            } else {
                return val
            }
        }
    }
}
</script>

<style scoped>
.dialog-footer {
    text-align: right;
}
.event-info {
}
.event-item {
    display: flex;
    flex-direction: row;
}
</style>
