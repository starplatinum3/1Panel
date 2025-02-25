<template>
    <div>
        <el-drawer
            v-model="drawerVisiable"
            :destroy-on-close="true"
            @close="handleClose"
            :close-on-click-modal="false"
            size="30%"
        >
            <template #header>
                <DrawerHeader :header="$t('ssh.pubkey')" :back="handleClose" />
            </template>
            <el-form ref="formRef" label-position="top" :rules="rules" :model="form" v-loading="loading">
                <el-row type="flex" justify="center">
                    <el-col :span="22">
                        <el-form-item :label="$t('ssh.encryptionMode')" prop="encryptionMode">
                            <el-select v-model="form.encryptionMode" @change="onLoadSecret">
                                <el-option label="ED25519" value="ed25519" />
                                <el-option label="ECDSA" value="ecdsa" />
                                <el-option label="RSA" value="rsa" />
                                <el-option label="DSA" value="dsa" />
                            </el-select>
                        </el-form-item>
                        <el-form-item :label="$t('commons.login.password')" prop="password">
                            <el-input v-model="form.password" type="password" show-password>
                                <template #append>
                                    <el-button @click="onCopy(form.password)" icon="DocumentCopy"></el-button>
                                    <el-button style="margin-left: 1px" @click="random" icon="RefreshRight"></el-button>
                                </template>
                            </el-input>
                        </el-form-item>

                        <el-form-item :label="$t('ssh.key')" prop="primaryKey" v-if="form.encryptionMode">
                            <el-input
                                v-model="form.primaryKey"
                                :autosize="{ minRows: 5, maxRows: 10 }"
                                type="textarea"
                            />
                            <div v-if="form.primaryKey">
                                <el-button icon="CopyDocument" class="margintop" @click="onCopy(form.primaryKey)">
                                    {{ $t('file.copy') }}
                                </el-button>
                                <el-button icon="Download" class="margintop" @click="onDownload">
                                    {{ $t('commons.button.download') }}
                                </el-button>
                            </div>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="drawerVisiable = false">{{ $t('commons.button.cancel') }}</el-button>
                    <el-button @click="onGenerate(formRef)" type="primary">
                        {{ $t('ssh.generate') }}
                    </el-button>
                </span>
            </template>
        </el-drawer>
    </div>
</template>
<script lang="ts" setup>
import { generateSecret, loadSecret } from '@/api/modules/host';
import { Rules } from '@/global/form-rules';
import i18n from '@/lang';
import { MsgError, MsgSuccess } from '@/utils/message';
import { dateFormatForName, getRandomStr } from '@/utils/util';
import useClipboard from 'vue-clipboard3';
import { FormInstance } from 'element-plus';
import DrawerHeader from '@/components/drawer-header/index.vue';
import { reactive, ref } from 'vue';
const { toClipboard } = useClipboard();

const loading = ref();
const drawerVisiable = ref();

const formRef = ref();
const form = reactive({
    password: '',
    encryptionMode: '',
    primaryKey: '',
});
const rules = reactive({
    encryptionMode: Rules.requiredSelect,
    password: [{ validator: checkPassword, trigger: 'blur' }],
});

function checkPassword(rule: any, value: any, callback: any) {
    if (form.password !== '') {
        const reg = /^[A-Za-z0-9]{6,15}$/;
        if (!reg.test(form.password)) {
            return callback(new Error(i18n.global.t('ssh.passwordHelper')));
        }
    }
    callback();
}

const acceptParams = async (): Promise<void> => {
    form.password = '';
    form.encryptionMode = 'rsa';
    form.primaryKey = '';
    onLoadSecret();
    drawerVisiable.value = true;
};

const random = async () => {
    form.password = getRandomStr(10);
};

const onLoadSecret = async () => {
    const res = await loadSecret(form.encryptionMode);
    form.primaryKey = res.data || '';
};

const onCopy = async (str: string) => {
    try {
        await toClipboard(str);
        MsgSuccess(i18n.global.t('commons.msg.copySuccess'));
    } catch (e) {
        MsgError(i18n.global.t('commons.msg.copyfailed'));
    }
};

const onGenerate = async (formEl: FormInstance | undefined) => {
    if (!formEl) return;
    formEl.validate(async (valid) => {
        if (!valid) return;
        let param = {
            encryptionMode: form.encryptionMode,
            password: form.password,
        };
        await generateSecret(param).then(() => {
            loading.value = false;
            MsgSuccess(i18n.global.t('commons.msg.operationSuccess'));
            onLoadSecret();
        });
    });
};
const onDownload = async () => {
    const downloadUrl = window.URL.createObjectURL(new Blob([form.primaryKey]));
    const a = document.createElement('a');
    a.style.display = 'none';
    a.href = downloadUrl;
    const href = window.location.href;
    const host = href.split('//')[1].split(':')[0];
    a.download = host + '_' + dateFormatForName(new Date()) + '_id_' + form.encryptionMode;
    const event = new MouseEvent('click');
    a.dispatchEvent(event);
};

const handleClose = () => {
    drawerVisiable.value = false;
};

defineExpose({
    acceptParams,
});
</script>

<style scoped lang="scss">
.margintop {
    margin-top: 10px;
}
</style>
