```ts
import { ref, type Ref } from 'vue'
import { CourseApi } from '@/services/apis'
import type { GlobalMaterial, CourseMaterial } from '@/store/material/types'
import type { MaterialReq, UploadMaterialReq, UploadMaterialRes, CourseMaterialRes, CourseUploadMaterialRes, deleteMaterialRes, CourseUpdateMaterialRes } from '@/services/material/types'
import { validFile, toastSuccess, toastError } from '@/utils/common/index'

type ChooseMessageFileRes = { tempFiles?: Array<{ name?: string; size?: number; type?: string; path: string }> }

export interface BaseState<T> {
    materials: Ref<T[]>
    isLoading: Ref<boolean>
    isRefreshing: Ref<boolean>
    hasMore: Ref<boolean>
    page: Ref<number>
    pageSize: Ref<number>
    totalCount: Ref<number>
    searchKeyword: Ref<string>
}

export interface BaseMethods {
    fetchMaterials: (reset?: boolean, name?: string) => Promise<void>
    handleSearch: () => Promise<void>
    loadMore: () => Promise<void>
    handleRefresh: () => Promise<void>
    handleUpload: () => Promise<void>
    handleDownload: ({ material }: { material: GlobalMaterial | CourseMaterial }) => Promise<void>
    handleOpen: ({ material }: { material: (GlobalMaterial | CourseMaterial) & { updatedAt?: string } }) => Promise<void>
    handleDelete: ({ id }: { id?: string }) => Promise<void>
    handleEdit: ({ id, name }: { id?: string; name: string }) => Promise<void>
}

export function useMaterials(mode: 'course', opts: { courseId: Ref<string> }): BaseState<CourseMaterial> & BaseMethods
export function useMaterials(mode: 'global'): BaseState<GlobalMaterial> & BaseMethods
export function useMaterials(mode: 'course' | 'global', opts?: { courseId?: Ref<string> }) {
    const searchKeyword = ref('')
    const isLoading = ref(false)
    const isRefreshing = ref(false)
    const hasMore = ref(true)
    const page = ref(1)
    const pageSize = ref(10)
    const totalCount = ref(0)

    const materials = ref<any[]>([]) as Ref<Array<GlobalMaterial | CourseMaterial>>

    const fetchMaterials = async (reset = false, name: string = '') => {
        try {
            if (reset) {
                page.value = 1
                hasMore.value = true
                materials.value = []
            }
            isLoading.value = true
            if (mode === 'course') {
                const courseId = opts?.courseId?.value || '';
                if (!courseId) {
                    toastError('课程ID缺失')
                    return
                }
                const res: CourseMaterialRes = await CourseApi.getCourseMaterial(courseId, { page: page.value, pageSize: pageSize.value, name } as MaterialReq)
                const data = res?.data
                const list = data?.list ?? []
                const count = data?.total ?? 0
                totalCount.value = count
                materials.value = materials.value.concat(list)
            } else {
                const res = await CourseApi.getMaterial({ page: page.value, pageSize: pageSize.value, name } as MaterialReq)
                const data = res?.data
                const list = data?.list || []
                const count = data?.total ?? 0
                totalCount.value = count
                materials.value = materials.value.concat(list)
            }
            const loaded = materials.value.length
            hasMore.value = loaded < totalCount.value
            if (hasMore.value) page.value += 1
        } catch (e) {
            toastError('加载失败')
        } finally {
            isLoading.value = false
        }
    }
    const refresh = async () => {
        isRefreshing.value = true
        await fetchMaterials(true)
        isRefreshing.value = false
    }

    const handleSearch = async () => {
        const keyword = (searchKeyword.value ?? '').trim()
        if (!keyword) {
            toastError('搜索内容不能为空')
            return
        }
        await fetchMaterials(true, keyword)
        toastSuccess('搜索成功')
    }

    const loadMore = async () => {
        if (hasMore.value && !isLoading.value) {
            await fetchMaterials(false)
            toastSuccess('加载成功')
        }
    }

    const handleRefresh = async () => {
        refresh()
        toastSuccess('刷新成功')
    }

    const handleUpload = async () => {
        const courseId = mode === 'course' ? String(opts?.courseId?.value || '') : ''
        if (mode === 'course' && !courseId) {
            toastError('课程ID缺失')
            return
        }
        uni.chooseMessageFile({
            count: 1,
            type: 'file',
            success: async (res: ChooseMessageFileRes) => {
                uni.showLoading({ title: '上传中，请稍等', mask: true })
                const fileInfo = res.tempFiles?.[0]
                if (!fileInfo) { uni.hideLoading(); return }
                const { name = '', size = 0, type, path } = fileInfo
                if (!validFile(name, type, size)) { uni.hideLoading(); return }
                try {
                    const req: UploadMaterialReq = { name, filePath: path }
                    if (mode === 'course') {
                        const upRes: CourseUploadMaterialRes = await CourseApi.uploadCourseMaterial(courseId, req)
                        console.log('上传课程')
                        if (upRes?.code === 0 || upRes?.data?.id) {
                            toastSuccess('上传成功')
                            refresh()
                        } else {
                            toastError('上传失败')
                        }
                    } else {
                        const upRes: UploadMaterialRes = await CourseApi.uploadMaterial(req)
                        console.log('上传全局')
                        if (upRes?.code === 0 || upRes?.data?.id) {
                            toastSuccess('上传成功')
                            refresh()
                        } else {
                            toastError(upRes?.data?.message || '上传失败')
                        }
                    }
                } catch {
                    toastError('上传失败')
                }
                uni.hideLoading()
            },
            fail: () => { toastError('选择文件失败'); uni.hideLoading() }
        })
    }

    const handleDownload = async ({ material }: { material: GlobalMaterial | CourseMaterial }) => {
        if (!material?.url) {
            toastError('无效的下载地址')
            return
        }
        uni.downloadFile({
            url: material.url as string,
            success: (res: { statusCode: number; tempFilePath?: string }) => {
                if (res.statusCode === 200 && res.tempFilePath) {
                    uni.saveFile({
                        tempFilePath: res.tempFilePath,
                        success: () => toastError('已保存到本地'),
                        fail: () => toastError('保存失败')
                    })
                } else {
                    toastError('下载失败')
                }
            },
            fail: () => toastError('下载失败')
        })
    }

    const handleOpen = async ({ material }: { material: (GlobalMaterial | CourseMaterial) & { updatedAt?: string } }) => {
        const cacheKey = `material_saved_${(material as any)?.id || material?.url}`
        const cached: { path: string; updatedAt?: string } | undefined = uni.getStorageSync(cacheKey)
        const downloadAndPersist = () => {
            if (!material?.url) {
                toastError('无效的下载地址')
                return
            }
            uni.downloadFile({
                url: material.url as string,
                success: (res: { statusCode: number; tempFilePath?: string }) => {
                    if (res.statusCode === 200 && res.tempFilePath) {
                        uni.saveFile({
                            tempFilePath: res.tempFilePath,
                            success: (s: { savedFilePath: string }) => {
                                uni.setStorage({ key: cacheKey, data: { path: s.savedFilePath, updatedAt: (material as any)?.updatedAt } })
                                uni.openDocument({ filePath: s.savedFilePath, showMenu: true })
                            },
                            fail: () => toastError('保存失败')
                        })
                    } else {
                        toastError('预览-下载失败')
                    }
                },
                fail: () => toastError('预览-下载失败')
            })
        }
        if (cached?.path) {
            uni.openDocument({
                filePath: cached.path,
                showMenu: true,
                success: () => { },
                fail: () => { uni.removeStorage({ key: cacheKey }); downloadAndPersist() }
            })
            return
        }
        downloadAndPersist()
    }

    const handleDelete = async ({ id }: { id?: string }) => {
        const finalId = String(id || '')
        if (!finalId) {
            toastError('ID 无效')
            return
        }
        uni.showModal({
            title: '确认删除',
            content: '确定要删除该资料吗？',
            success: async (res) => {
                if (res.confirm) {
                    try {
                        if (mode === 'course') {
                            const courseId = String(opts?.courseId?.value || '')
                            if (!courseId) { toastError('课程ID缺失'); return }
                            const resp: deleteMaterialRes = await CourseApi.deleteCourseMaterial(courseId, finalId)
                            if ((resp as any)?.code === 0) {
                                toastSuccess('删除成功');
                                refresh()
                            } else { toastError('删除失败') }
                        } else {
                            const resp: deleteMaterialRes = await CourseApi.deleteMaterial(finalId)
                            if ((resp as any)?.code === 0) {
                                toastSuccess('删除成功');
                                refresh()
                            } else { toastError('删除失败') }
                        }
                    } catch {
                        toastError('删除失败')
                    }
                }
            }
        })
    }

    const handleEdit = async ({ id, name }: { id?: string; name: string }) => {
        const finalId = String(id || '').trim()
        const finalName = String(name || '').trim()
        if (!finalId) { toastError('ID 无效'); return }
        if (!finalName) { toastError('文件名不能为空'); return }
        try {
            if (mode === 'course') {
                const courseId = String(opts?.courseId?.value || '')
                if (!courseId) { toastError('课程ID缺失'); return }
                const res: CourseUpdateMaterialRes = await CourseApi.updateCourseMaterial(courseId, finalId, { name: finalName })
                if (res?.code === 0) {
                    toastSuccess('更新成功');
                    refresh()
                } else { toastError('更新失败') }
            } else {
                const res = await CourseApi.updateMaterial(finalId, { name: finalName })
                if ((res as any)?.code === 0) {
                    toastSuccess('更新成功');
                    refresh()
                } else { toastError('更新失败') }
            }
        } catch {
            toastError('更新异常')
        }
    }

    return {
        // state
        materials,
        isLoading,
        isRefreshing,
        hasMore,
        page,
        pageSize,
        totalCount,
        searchKeyword,
        // methods
        fetchMaterials,
        handleSearch,
        loadMore,
        handleRefresh,
        handleUpload,
        handleDownload,
        handleOpen,
        handleDelete,
        handleEdit,
    }
}

```