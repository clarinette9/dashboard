<!-- eslint-disable -->
<template>
    <div>
        <Button
            class="add-button"
            type="primary"
            size="small"
            @click="onAdd"
        >{{ $t('com.createX', {obj: $t('user.name')}) }}</Button>
        <Drawer
            v-model="isHiden"
            :title="handleType === 'create' ? ($t('com.createX', {obj: $t('user.name')})) :
            $t('user.changePassword')"
            width="30"
            :mask-closable="false"
        >
            <CreateUser
                v-if="isHiden && handleType === 'create'"
                @submit="submitData"
            />
            <UpdatePassword
                v-if="isHiden && handleType === 'updatePassword'"
                :currentData="currentData"
                @submit="submitPassword"
                @cancel="isHiden = false"
            />
        </Drawer>
        <pageTable
            ref="pageTable"
            :tableData="tableData"
            :columns="columns"
            :loading="loading"
        />
    </div>
</template>

<script>
import pageTable from '@/components/table/pageTable';
import CreateUser from './CreateUser.vue';
import UpdatePassword from './UpdatePassword.vue';
    export default {
        components: {
            pageTable,
            CreateUser,
            UpdatePassword
        },
        mounted() {
            this.getUserList();
        },
        data() {
            const that = this;
            return {
                loading: false,
                isHiden: false,
                handleType: '',
                currentData: {},
                tableData: [],
                associatedUseList: [],
                columns: [
                    {
                        key: 'user_name',
                        title: this.$t('user.name'),
                        searchable: true,
                        sortable: 'custom',
                    },
                    {
                        key: 'is_admin',
                        title: this.$t('user.role'),
                        render(h, params) {
                            return (
                                <div>
                                    {
                                        <span>{params.row.is_admin ? that.$t('com.sysAdmin') : that.
                                        $t('com.ordinaryUser')}</span>
                                    }
                                </div>
                            );
                        }
                    },
                    {
                        key: 'action',
                        title: this.$t('com.operation'),
                        render(h, params) {
                            return h('div', [
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '10px'
                                    },
                                    on: {
                                        click: () => {
                                            that.onUpdatePassword(params);
                                        }
                                    }
                                }, that.$t('user.changePassword')),
                                h('Button', {
                                    props: {
                                        type: 'error',
                                        size: 'small'
                                    },
                                    on: {
                                        click: () => {
                                           that.onDel(params);
                                        }
                                    }
                                }, that.$t('com.del')),
                            ]
                            );
                        }
                    }
                ]
            };
        },
        methods: {
            submitPassword(data, self) {
                this.upsertPassWord(data, self);
            },
            upsertPassWord(data, self) {
                let tmpData = {};
                if (self) {
                    tmpData = {
                        old_password: data.old_password,
                        password: data.new_password
                    };
                }
                else {
                    tmpData = {
                        password: data.new_password
                    };
                }
                this.$request({
                    url: this.$urlFormat('auth/users/{user_name}/passwd', {user_name: data.user_name}),
                    method: 'patch',
                    data: tmpData
                }).then(data => {
                    if (data.status === 200) {
                        this.$Message.success({
                            content: this.$t('com.tipEditSuccX', {obj: this.$t('com.password')})
                        });
                        this.isHiden = false;
                        if (self) {
                            this.$Modal.warning({
                                title: this.$t('com.tips'),
                                content: this.$t('user.tipPasswordModified'),
                                onOk: () => {
                                    location.href = `${window.location.protocol}//${window.location.host}/login`;
                                }
                            });
                        }
                    }
                });
            },
            onDel(params) {
                this.$Modal.confirm({
                    title: this.$t('com.informationTips'),
                    content: this.$t('com.confirmDelX', {obj: this.$t('user.name')}) + params.row.user_name + '?',
                    onOk: () => {
                        this.onDelAdminUser(params.row.user_name);
                    }
                });
            },
            onDelAdminUser(name) {
                let user = this.$store.getUser();
                this.$request({
                    url: this.$urlFormat('auth/users/{user_name}', {
                        user_name: name
                    }),
                    method: 'delete'
                }).then(data => {
                    if (data.status === 200) {
                        if (user.name === name) {
                            this.$Modal.error({
                                loading: true,
                                title: this.$t('com.informationTips'),
                                content: this.$t('user.tipUserDel'),
                                onOk: () => {
                                    this.$Modal.remove();
                                    location.href = `${window.location.protocol}//${window.location.host}/login`;
                                },
                            });
                        }
                        else {
                            this.getUserList();
                        }
                    }
                });
            },
            submitData(data) {
                if (data) {
                    this.onAddAdminUser(data);
                }
            },
            onAddAdminUser(data) {
                this.$request({
                    url: 'auth/users',
                    method: 'post',
                    data: data
                }).then(data => {
                    if (data.status === 200) {
                        this.$Message.success({content: this.$t('com.tipAddSucc')});
                        this.isHiden = false;
                        this.getUserList();
                    }
                });
            },
            getUserList() {
                this.loading = true;
                this.$request({
                    url: 'auth/users',
                    method: 'get'
                }).then(data => {
                    if (data.status === 200) {
                        this.tableData = data.data.Data;
                    }
                }).finally(() => {
                    this.loading = false;
                });
            },
            onAdd() {
                this.isHiden = true;
                this.handleType = 'create';
            },
            onUpdatePassword(params) {
                this.isHiden = true;
                this.handleType = 'updatePassword';
                this.currentData = params.row;
            }
        }
    };
</script>

<style lang="less" scoped>

</style>