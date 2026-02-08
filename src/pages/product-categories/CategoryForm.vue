<script setup lang="ts">
import {
    createCategory,
    getCategory,
    updateCategory,
    uploadCategoryImage,
} from '@/api/product-categories.api'
import router from '@/router'
import {
    Button,
    FileUpload,
    InputText,
    Message,
    Textarea,
    useToast,
} from 'primevue'
import { computed, onMounted, ref } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const toast = useToast()

const loading = ref(false)
const errors = ref<Record<string, string[]>>({})

const form = ref({
    id: 0,
    name: '',
    description: '',
})

const categoryId = computed<number | null>(() =>
    route.params.id ? Number(route.params.id) : null
)

const isEdit = computed(() => !!categoryId.value)

const selectedFile = ref<File | null>(null)
const imagePreview = ref<string | null>(null)

const onFileSelect = (event: { files: File[] }) => {
    const file = event.files[0]

    if (!file) return

    selectedFile.value = file
    imagePreview.value = URL.createObjectURL(file)
}

const submit = async () => {
    loading.value = true

    try {
        if (isEdit.value) {
            await updateCategory(form.value.id, form.value)
        } else {
            const res = await createCategory(form.value)
            form.value.id = res.data.data.id
        }

        if (selectedFile.value) {
            const fd = new FormData()
            fd.append('image', selectedFile.value)
            await uploadCategoryImage(form.value.id, fd)
        }

        toast.add({
            severity: 'success',
            summary: 'Success',
            detail: isEdit.value
                ? 'Category updated succesfully'
                : 'Category created successfuly',
            life: 3000,
        })

        router.push('/product-categories')
    } catch (error: any) {
        if (error.response?.status === 422) {
            errors.value = error.response?.data.errors ?? {}
            return
        }

        toast.add({
            severity: 'error',
            summary: 'Error',
            detail: error.response?.data?.message,
            life: 3000,
        })
    } finally {
        loading.value = false
    }
}

onMounted(async () => {
    if (!isEdit.value) return

    loading.value = true

    try {
        const res = await getCategory(categoryId.value!)
        const data = res.data.data

        form.value.id = data.id
        form.value.name = data.name
        form.value.description = data.description ?? ''
        imagePreview.value = data.image ?? ''
    } catch (error) {
        toast.add({
            severity: 'error',
            summary: 'Error',
            detail: 'Failed to load category',
            life: 3000,
        })
    } finally {
        loading.value = false
    }
})
</script>

<template>
    <div class="min-h-screen bg-surface-50 font-sans text-surface-900">
        <div class="mb-8 flex items-center justify-between">
            <div>
                <h1 class="mb-1 text-2xl font-bold text-surface-900">
                    <span v-if="isEdit"> Edit Category </span>
                    <span v-else> Create Category </span>
                </h1>
                <p class="text-sm text-surface-500">
                    <span v-if="isEdit"> Update caetgory detail </span>
                    <span v-else> Add a new product category </span>
                </p>
            </div>

            <Button asChild v-slot="slotProps">
                <RouterLink
                    :to="{ name: 'product-categories' }"
                    :class="slotProps.class"
                >
                    <i class="pi pi-arrow-left"></i>
                    Back
                </RouterLink>
            </Button>
        </div>

        <div
            class="rouned-2xl overflow-hidden border border-surface-200 bg-white"
        >
            <form @submit.prevent="submit">
                <div
                    class="grid grid-cols-1 gap-8 p-6 md:grid-cols-12 md:gap-12 md:p-8"
                >
                    <div class="flex flex-col gap-4 md:col-span-4">
                        <label class="text-sm font-semibold text-surface-900"
                            >Category Image</label
                        >

                        <div
                            class="group relative flex aspect-square w-full flex-col items-center justify-center overflow-hidden rounded-xl border-2 border-dashed border-surface-200 bg-surface-50 transition-colors hover:border-primary-500"
                        >
                            <img
                                v-if="imagePreview"
                                :src="imagePreview"
                                class="h-full w-full object-cover"
                            />

                            <div
                                v-else
                                class="flex flex-col items-center gap-2 text-surface-400"
                            >
                                <i class="pi pi-image text-4xl opacity-50"></i>
                                <span class="text-sm font-medium"
                                    >No image selected</span
                                >
                            </div>
                        </div>

                        <div class="flex flex-col gap-2">
                            <FileUpload
                                mode="basic"
                                @select="onFileSelect"
                                :auto="false"
                                accept="image/*"
                                :maxFileSize="2000000"
                                chooseLabel="Choose Image"
                                class="w-full"
                            />
                            <small class="text-center text-xs text-surface-500"
                                >Max size: 2MB. Formats: JPG, PNG.</small
                            >
                        </div>
                    </div>

                    <div class="flex flex-col gap-6 md:col-span-8">
                        <div class="flex flex-col gap-2">
                            <label
                                for="name"
                                class="font-medium text-surface-900"
                            >
                                Name <span class="text-red-600">*</span>
                            </label>

                            <InputText
                                id="name"
                                v-model="form.name"
                                type="text"
                                placeholder="Electronics.."
                                fluid
                                class="border-surface-200 bg-surface-50! focus:bg-white!"
                                :invalid="!!errors.name"
                            />

                            <Message
                                v-if="errors.name"
                                severity="error"
                                size="small"
                                variant="simple"
                            >
                                {{ errors.name[0] }}
                            </Message>
                        </div>

                        <div class="flex flex-col gap-2">
                            <label
                                for="description"
                                class="font-medium text-surface-900"
                            >
                                Description
                            </label>

                            <Textarea
                                id="description"
                                v-model="form.description"
                                type="text"
                                placeholder="Electronics.."
                                fluid
                                class="border-surface-200 bg-surface-50! focus:bg-white!"
                                rows="6"
                            />
                        </div>

                        <div
                            class="mt-auto flex justify-end border-t border-surface-100 pt-4"
                        >
                            <div class="flex gap-3">
                                <Button
                                    label="Cancel"
                                    severity="secondary"
                                    text
                                    @click="router.back()"
                                ></Button>
                                <Button
                                    type="submit"
                                    label="Save Category"
                                    icon="pi pi-check"
                                ></Button>
                            </div>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </div>
</template>
