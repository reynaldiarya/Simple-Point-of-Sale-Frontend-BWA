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
import { useCustomerStore } from '@/stores/customer.store'
import { storeToRefs } from 'pinia'
import { onMounted } from 'vue'
import { useDebounceFn } from '@vueuse/core'
import { deleteCustomer } from '@/api/customers.api'

const customerStore = useCustomerStore()
const { fetch, setLimit, setPage, nextPage, prevPage } = customerStore
const { items, loading, limit, currentPage, totalPages, search } =
    storeToRefs(customerStore)

const confirm = useConfirm()
const toast = useToast()

const onSearch = useDebounceFn(() => {
    setPage(1)
}, 400)

const confirmDelete = (id: number) => {
    confirm.require({
        message: 'Are you sure you want to delete this customer?',
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
                await deleteCustomer(id)
                toast.add({
                    severity: 'success',
                    summary: 'Deleted',
                    detail: 'Customer Removed',
                    life: 3000,
                })
                fetch()
            } catch (error) {
                toast.add({
                    severity: 'error',
                    summary: 'Error',
                    detail: 'Failed to delete customer',
                    life: 3000,
                })
            }
        },
    })
}

onMounted(() => {
    fetch()
})
</script>

<template>
    <div class="min-h-screen bg-surface-50 font-sans text-surface-900">
        <div class="mb-8 flex items-center justify-between">
            <div>
                <h1 class="mb-1 text-2xl font-bold text-surface-900">
                    Customers
                </h1>
                <p class="text-sm text-surface-500">
                    The list here shows all customers
                </p>
            </div>

            <Button asChild v-slot="slotProps">
                <RouterLink
                    :to="{ name: 'customers-create' }"
                    :class="slotProps.class"
                >
                    Add Customer
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
                            <div
                                class="flex h-10 w-10 items-center justify-center rounded-full bg-primary-100"
                            >
                                <i class="pi pi-user text-primary-600"></i>
                            </div>
                            <span class="font-semibold text-surface-900">
                                {{ data.name }}
                            </span>
                        </div>
                    </template>
                </Column>

                <Column field="phone" header="Phone">
                    <template #body="{ data }">
                        <span class="text-surface-600">
                            {{ data.phone }}
                        </span>
                    </template>
                </Column>

                <Column header="Actions" style="width: 5rem">
                    <template #body="{ data }">
                        <div class="flex items-center gap-2">
                            <RouterLink
                                :to="{
                                    name: 'customers-edit',
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
