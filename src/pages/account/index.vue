<template>
    <v-container>
        <div class="mb-2">
            <v-card class="mx-auto" :disabled="UserCardLoading" :loading="UserCardLoading">
                <template v-slot:prepend>
                    <v-avatar class="mb-2">
                        <v-img :alt="localuserinfo.display_name"
                            :src="'https://s4-1.wuyuan.1r.ink/user/' + localuserinfo.avatar"></v-img>
                    </v-avatar>
                </template>
                <template v-slot:title>
                    <span class="font-weight-black">{{
                        localuserinfo.display_name
                    }}</span>
                </template>
                <template v-slot:loader="{ isActive }">
                    <v-progress-linear :active="isActive" height="4" indeterminate></v-progress-linear>
                </template>
                <template v-slot:subtitle>
                    <v-chip>
                        <v-icon icon="mdi-account-circle" start></v-icon>

                        ID:{{ localuserinfo.userid }}
                    </v-chip>
                    <v-chip>
                        <v-icon icon="mdi-tag" start></v-icon>

                        创作者</v-chip>
                    <span class=""></span>
                </template>

                <v-card-text class="bg-surface-light pt-4">
                    {{ UserInfo }}
                </v-card-text>
            </v-card>
        </div>
        <v-card>
            <v-tabs v-model="tab" bg-color="primary">
                <v-tab value="userinfo">用户信息</v-tab>
                <v-tab value="username">用户名</v-tab>
                <v-tab value="password">密码</v-tab>
            </v-tabs>

            <v-card-text>
                <v-tabs-window v-model="tab">
                    <v-tabs-window-item value="userinfo">
                        <v-form v-model="valid">
                            <v-container>
                                <v-row>
                                    <v-col cols="12" md="4">
                                        <v-text-field v-model="UserInfo.display_name" :counter="10" label="显示名称"
                                            required></v-text-field>
                                    </v-col>

                                    <v-col cols="12" md="12">
                                        <v-textarea label="显示简介" v-model="UserInfo.motto" :counter="500"></v-textarea> </v-col><v-col
                                        cols="12" md="4">
                                        <v-select v-model="select" :items="items" item-title="state" item-value="abbr"
                                            label="性别" persistent-hint return-object></v-select></v-col>
                                </v-row><v-btn @click="submit" :disabled="!valid"> 提交 </v-btn>
                            </v-container>
                        </v-form>
                    </v-tabs-window-item>

                    <v-tabs-window-item value="username"><v-form v-model="usernamevalid">
                            <v-container>
                                <v-row>
                                    <v-col cols="12" md="4">
                                        <v-text-field v-model="UserInfo.username" :counter="10" :rules="nameRules"
                                            label="用户名" required></v-text-field>
                                    </v-col>


                                </v-row><v-btn @click="changeusername" :disabled="!usernamevalid"> 提交 </v-btn>
                            </v-container>
                        </v-form></v-tabs-window-item>

                    <v-tabs-window-item value="password"> <v-form v-model="passwordvalid">
                            <v-container>
                                <v-row>
                                    <v-col cols="12" md="8">
                                        <v-text-field v-model="oldpassword" hint="此框不验证输入值" label="原密码"
                                            required></v-text-field>
                                    </v-col>
                                    <v-col cols="12" md="8">
                                        <v-text-field v-model="newpassword" :rules="passwordRules" label="新密码"
                                            required></v-text-field>
                                    </v-col>

                                </v-row><v-btn @click="changepassword" :disabled="!passwordvalid"> 提交 </v-btn>
                            </v-container>
                        </v-form></v-tabs-window-item>
                </v-tabs-window>
            </v-card-text>
        </v-card>
    </v-container>
</template>

<script>
import request from "../../axios/axios";
import {localuser} from '@/stores/user';


import "@waline/client/style";
export default {
    data() {
        return {
            UserCardLoading: false,
            localuserinfo: localuser.user,
            UserInfo: {},
            tab: null,
            select: { state: "未知", abbr: "3" },
            items: [
                { state: "男", abbr: "0" },
                { state: "女", abbr: "1" },
                { state: "猫娘", abbr: "2" },
                { state: "未知", abbr: "3" },
            ],

            valid: false,
            nameRules: [
                (value) => {
                    if (value) return true;

                    return "Name is required.";
                },
                (value) => {
                    if (value?.length <= 10) return true;

                    return "Name must be less than 10 characters.";
                },
                (value) => {
                    const regex = /^[a-z]+$/
                    if (regex.test(value)) return true

                    return '仅小写字母'
                },
            ],
            passwordRules: [
                (value) => {
                    if (value) return true;

                    return "password is required.";
                },
                (value) => {
                    if (value?.length <= 10) return true;

                    return "Name must be less than 10 characters.";
                },
                (value) => {
                    const regex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[$@$!%*?&\.])[A-Za-z\d$@$!%*?&\.]{8,}/
                    if (regex.test(value)) return true

                    return '密码必须由数字、大小写字母和特殊字符组成，且长度至少为8位';
                },
            ],
            oldpassword: '',
            newpassword: '',
            passwordvalid: false,
            usernamevalid: false,
        };
    },

    async created() {
        await this.getuserinfo();
    },
    methods: {
        async getuserinfo() {
            this.UserInfo = await request({
                url: "/api/getuserinfo?id=" + this.localuserinfo.userid,
                method: "get",
            });
            this.UserInfo = this.UserInfo.info.user;
            this.select = this.items.find((item) => item.abbr == this.UserInfo.sex);
            this.UserCardLoading = false;
            console.log(this.UserInfo);
        },
        async submit() {
            this.UserCardLoading = true;
            const response = await request({
                url: "/my/set/userinfo",
                method: "post",
                data: {
                    display_name: this.UserInfo.display_name,
                    aboutme: this.UserInfo.motto,
                    sex: this.select.abbr,
                },
            });
            console.log(response.status);
            this.$toast.add({ severity: 'info', summary: '修改个人信息', detail: response.status, life: 3000 });

            await this.getuserinfo();
            this.UserCardLoading = false;

        },
        async changeusername() {
            this.UserCardLoading = true;
            const response = await request({
                url: "/my/set/username",
                method: "post",
                data: {
                    username: this.UserInfo.username,
                },
            });
            this.$toast.add({ severity: 'info', summary: '修改用户名', detail: response.status, life: 3000 });

            console.log(response.status);

            await this.getuserinfo();
            this.UserCardLoading = false;
        },
        async changepassword() {
            this.UserCardLoading = true;
            const response = await request({
                url: "/my/set/pw",
                method: "post",
                data: {
                    oldpw: this.oldpassword, newpw: this.newpassword
                },
            });
            console.log(response.status);
            this.$toast.add({ severity: 'info', summary: '修改密码', detail: response.message, life: 3000 });

            await this.getuserinfo();
            this.UserCardLoading = false;
        },
    },
};
</script>
