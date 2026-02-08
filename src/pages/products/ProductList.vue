<script setup lang="ts">
import {
    Button,
    Column,
    ConfirmDialog,
    DataTable,
    IconField,
    InputIcon,
    InputText,
    Select,
    useConfirm,
    useToast,
} from 'primevue'
import { useProductStore } from '@/stores/product.store'
import { useProductCategoryStore } from '@/stores/product-category.store'
import { storeToRefs } from 'pinia'
import { computed, onMounted } from 'vue'
import { useDebounceFn } from '@vueuse/core'
import { deleteProduct } from '@/api/products.api'

const productStore = useProductStore()
const { fetch, setLimit, setPage, nextPage, prevPage, setCategoryId } =
    productStore
const { items, loading, limit, currentPage, totalPages, search, categoryId } =
    storeToRefs(productStore)

const categoryStore = useProductCategoryStore()
const { items: categories } = storeToRefs(categoryStore)

const confirm = useConfirm()
const toast = useToast()

const categoryOptions = computed(() => [
    { label: 'All Categories', value: null },
    ...categories.value.map((c) => ({ label: c.name, value: c.id })),
])

const formatPrice = (price: number) => {
    return new Intl.NumberFormat('id-ID', {
        style: 'currency',
        currency: 'IDR',
        minimumFractionDigits: 0,
    }).format(price)
}

const onSearch = useDebounceFn(() => {
    setPage(1)
}, 400)

const onCategoryChange = (value: number | null) => {
    setCategoryId(value)
}

const confirmDelete = (id: number) => {
    confirm.require({
        message: 'Are you sure you want to delete this product?',
        header: 'Confirm Delete',
        icon: 'pi pi-exclamation-triangle',
        rejectProps: {
            label: 'Cancel',
            severity: 'secondary',
            outlined: true,
        },
        acceptProps: {
            label: 'Delete',
            severity: 'danger',
        },
        accept: async () => {
            try {
                await deleteProduct(id)
                toast.add({
                    severity: 'success',
                    summary: 'Deleted',
                    detail: 'Product Removed',
                    life: 3000,
                })
                fetch()
            } catch (error) {
                toast.add({
                    severity: 'error',
                    summary: 'Error',
                    detail: 'Failed to delete product',
                    life: 3000,
                })
            }
        },
    })
}

onMounted(() => {
    fetch()
    categoryStore.fetch()
})
</script>

<template>
    <div class="min-h-screen bg-surface-50 font-sans text-surface-900">
        <div class="mb-8 flex items-center justify-between">
            <div>
                <h1 class="mb-1 text-2xl font-bold text-surface-900">
                    Products
                </h1>
                <p class="text-sm text-surface-500">
                    The list here shows all products
                </p>
            </div>

            <Button asChild v-slot="slotProps">
                <RouterLink
                    :to="{ name: 'products-create' }"
                    :class="slotProps.class"
                >
                    Add Product
                </RouterLink>
            </Button>
        </div>

        <div class="rounded-2xl border border-surface-200 bg-white p-2">
            <div
                class="flex flex-col items-center justify-between gap-4 px-4 py-4 md:flex-row"
            >
                <IconField iconPosition="left" class="w-full md:w-80">
                    <InputIcon class="pi pi-search text-surface-400" />
                    <InputText
                        v-model="search"
                        placeholder="Search"
                        @input="onSearch"
                    />
                </IconField>

                <Select
                    :model-value="categoryId"
                    :options="categoryOptions"
                    optionLabel="label"
                    optionValue="value"
                    placeholder="Filter by Category"
                    class="w-full md:w-56"
                    filter
                    filterPlaceholder="Search categories..."
                    @update:model-value="onCategoryChange"
                />
            </div>

            <DataTable
                :value="items"
                :loading="loading"
                dataKey="id"
                class="clean-table"
                :rowHover="true"
            >
                <Column field="name" header="Name" class="min-w-[16rem]">
                    <template #body="{ data }">
                        <div class="flex items-center gap-3">
                            <div class="relative">
                                <img
                                    :src="data.image ?? '/placeholder.png'"
                                    class="h-10 w-10 rounded-full bg-surface-100 object-cover"
                                />
                            </div>
                            <span class="font-semibold text-surface-900">
                                {{ data.name }}
                            </span>
                        </div>
                    </template>
                </Column>

                <Column field="price" header="Price">
                    <template #body="{ data }">
                        <span class="font-medium text-surface-900">
                            {{ formatPrice(data.price) }}
                        </span>
                    </template>
                </Column>

                <Column field="stock" header="Stock">
                    <template #body="{ data }">
                        <span
                            :class="[
                                'rounded-full px-2 py-1 text-xs font-medium',
                                data.stock > 10
                                    ? 'bg-green-100 text-green-700'
                                    : data.stock > 0
                                      ? 'bg-yellow-100 text-yellow-700'
                                      : 'bg-red-100 text-red-700',
                            ]"
                        >
                            {{ data.stock }}
                        </span>
                    </template>
                </Column>

                <Column field="category" header="Category">
                    <template #body="{ data }">
                        <span class="text-surface-600">
                            {{ data.category?.name ?? '-' }}
                        </span>
                    </template>
                </Column>

                <Column header="Actions" style="width: 5rem">
                    <template #body="{ data }">
                        <div class="flex items-center gap-2">
                            <RouterLink
                                :to="{
                                    name: 'products-edit',
                                    params: { id: data.id },
                                }"
                            >
                                <Button
                                    icon="pi pi-pencil"
                                    text
                                    rounded
                                    severity="primary"
                                    class="h-9! w-9! border-surface-200! bg-white text-surface-200! hover:border-primary-500 hover:bg-primary-50! hover:text-primary-600!"
                                />
                            </RouterLink>

                            <Button
                                icon="pi pi-trash"
                                text
                                rounded
                                severity="danger"
                                class="h-9! w-9! border-surface-200! bg-white text-surface-200! hover:border-primary-500 hover:bg-primary-50! hover:text-primary-600!"
                                @click="confirmDelete(data.id)"
                            />
                        </div>
                    </template>
                </Column>
            </DataTable>

            <div
                class="flex items-center justify-between gap-4 border-t border-surface-100 px-4 py-4"
            >
                <div class="flex items-center gap-2">
                    <span class="text-sm text-surface-500">Rows per page:</span>
                    <Select
                        :model-value="limit"
                        :options="[5, 10, 20, 50]"
                        @update:model-value="setLimit"
                    />
                </div>

                <div class="flex items-center gap-4">
                    <span class="text-sm font-medium text-surface-600">
                        {{ currentPage }} of {{ totalPages }}
                    </span>

                    <div class="flex gap-1">
                        <Button
                            icon="pi pi-chevron-left"
                            text
                            rounded
                            severity="secondary"
                            :disabled="currentPage === 1"
                            class="h-9! w-9! border! border-surface-500! hover:bg-surface-50!"
                            @click="prevPage()"
                        />
                        <Button
                            icon="pi pi-chevron-right"
                            text
                            rounded
                            severity="secondary"
                            :disabled="currentPage === totalPages"
                            class="h-9! w-9! border! border-surface-500! hover:bg-surface-50!"
                            @click="nextPage()"
                        />
                    </div>
                </div>
            </div>
        </div>
    </div>

    <ConfirmDialog />
</template>
