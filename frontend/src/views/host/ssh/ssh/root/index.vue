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
                <DrawerHeader :header="$t('ssh.permitRootLogin')" :back="handleClose" />
            </template>
            <el-form ref="formRef" label-position="top" :model="form" @submit.prevent v-loading="loading">
                <el-row type="flex" justify="center">
                    <el-col :span="22">
                        <el-form-item :label="$t('ssh.permitRootLogin')" prop="permitRootLogin">
                            <el-select v-model="form.permitRootLogin" style="width: 100%">
                                <el-option :label="$t('ssh.rootHelper1')" value="yes" />
                                <el-option :label="$t('ssh.rootHelper2')" value="no" />
                                <el-option :label="$t('ssh.rootHelper3')" value="without-password" />
                                <el-option :label="$t('ssh.rootHelper4')" value="forced-commands-only" />
                            </el-select>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="drawerVisiable = false">{{ $t('commons.button.cancel') }}</el-button>
                    <el-button :disabled="loading" type="primary" @click="onSave(formRef)">
                        {{ $t('commons.button.confirm') }}
                    </el-button>
                </span>
            </template>
        </el-drawer>
    </div>
</template>
<script lang="ts" setup>
import { reactive, ref } from 'vue';
import i18n from '@/lang';
import { MsgSuccess } from '@/utils/message';
import { ElMessageBox, FormInstance } from 'element-plus';
import { updateSSH } from '@/api/modules/host';
import DrawerHeader from '@/components/drawer-header/index.vue';

const emit = defineEmits<{ (e: 'search'): void }>();

interface DialogProps {
    permitRootLogin: string;
}
const drawerVisiable = ref();
const loading = ref();

const form = reactive({
    permitRootLogin: 'yes',
});

const formRef = ref<FormInstance>();

const acceptParams = (params: DialogProps): void => {
    form.permitRootLogin = params.permitRootLogin;
    drawerVisiable.value = true;
};

const onSave = async (formEl: FormInstance | undefined) => {
    if (!formEl) return;
    formEl.validate(async (valid) => {
        if (!valid) return;
        ElMessageBox.confirm(
            i18n.global.t('ssh.sshChangeHelper', [
                i18n.global.t('ssh.permitRootLogin'),
                loadPermitLabel(form.permitRootLogin),
            ]),
            i18n.global.t('ssh.sshChange'),
            {
                confirmButtonText: i18n.global.t('commons.button.confirm'),
                cancelButtonText: i18n.global.t('commons.button.cancel'),
                type: 'info',
            },
        )
            .then(async () => {
                loading.value = true;
                await updateSSH('PermitRootLogin', form.permitRootLogin)
                    .then(() => {
                        loading.value = false;
                        handleClose();
                        emit('search');
                        MsgSuccess(i18n.global.t('commons.msg.operationSuccess'));
                    })
                    .catch(() => {
                        loading.value = false;
                    });
            })
            .catch(() => {
                emit('search');
            });
    });
};

const loadPermitLabel = (value: string) => {
    switch (value) {
        case 'yes':
            return i18n.global.t('ssh.rootHelper1');
        case 'no':
            return i18n.global.t('ssh.rootHelper2');
        case 'without-password':
            return i18n.global.t('ssh.rootHelper3');
        case 'forced-commands-only':
            return i18n.global.t('ssh.rootHelper4');
    }
};

const handleClose = () => {
    drawerVisiable.value = false;
};

defineExpose({
    acceptParams,
});
</script>
