<template>
  <div class="detail-container" v-if="student">
    <div class="page-header">
      <div class="header-content">
        <div class="header-info">
          <h1 class="page-title">
            {{ isCreatingNew ? '新增学生档案' : (viewMode === 'admin' ? editableStudent.name : '我的个人档案') }}
          </h1>
          <p class="page-subtitle">
            {{ isCreatingNew ? '请完善以下学生信息' :
              (viewMode === 'admin' ? `学号：${editableStudent.studentId}` :
                  `${student.name} (${student.studentId})`) }}
          </p>

          <div class="status-badges" v-if="!isCreatingNew">
            <span class="status-badge" :class="getStatusClass(student.studentStatus)">
              {{ student.studentStatus }}
            </span>
            <span class="info-badge" v-if="student.academicYear">
              第{{ student.academicYear }}学年
            </span>
            <span class="info-badge" v-if="student.semester">
              {{ formatSemester(student.semester) }}
            </span>
          </div>
        </div>

        <div class="header-actions">
          <template v-if="viewMode === 'admin'">
            <button v-if="!isEditing && !isCreatingNew" class="action-btn edit-btn" @click="startEditing">
              编辑档案
            </button>
            <button v-if="!isEditing && !isCreatingNew" class="action-btn delete-btn" @click="handleDelete">
              删除档案
            </button>
            <button v-if="isEditing" class="action-btn save-btn" @click="handleSave" :disabled="isSaving">
              <div v-if="isSaving" class="loading-spinner"></div>
              <span>{{ isSaving ? '保存中...' : (isCreatingNew ? '创建档案' : '保存更改') }}</span>
            </button>
            <button v-if="isEditing && !isCreatingNew" class="action-btn cancel-btn" @click="cancelEditing">
              取消编辑
            </button>
          </template>

          <template v-else-if="viewMode === 'student'">
            <button v-if="!isEditing" class="action-btn edit-btn" @click="startEditing">
              修改信息
            </button>
            <button v-if="isEditing" class="action-btn save-btn" @click="handleSave" :disabled="isSaving">
              <div v-if="isSaving" class="loading-spinner"></div>
              <span>{{ isSaving ? '保存中...' : '保存更改' }}</span>
            </button>
            <button v-if="isEditing" class="action-btn cancel-btn" @click="cancelEditing">
              取消
            </button>
          </template>
        </div>
      </div>
    </div>

    <div class="content-wrapper">
      <div class="content-grid">
        <div class="info-card">
          <div class="card-header">
            <h4>基本信息</h4>
          </div>
          <div class="card-content">
            <div class="form-grid">
              <div class="info-item">
                <label class="item-label">姓名</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.name }}</span>
                  <input v-else v-model="editableStudent.name" class="form-input" :class="{'is-invalid': errors.name}" :disabled="viewMode === 'student'" placeholder="请输入姓名"/>
                  <span v-if="errors.name" class="error-message">{{ errors.name }}</span>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">性别</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.gender }}</span>
                  <select v-else v-model="editableStudent.gender" class="form-select" :class="{'is-invalid': errors.gender}" :disabled="viewMode === 'student'">
                    <option value="男">男</option>
                    <option value="女">女</option>
                    <option value="其他">其他</option>
                  </select>
                  <span v-if="errors.gender" class="error-message">{{ errors.gender }}</span>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">出生日期</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ formatDate(student.dateOfBirth) }}</span>
                  <input v-else type="date" v-model="editableStudent.dateOfBirth" class="form-input" :class="{'is-invalid': errors.dateOfBirth}" :disabled="viewMode === 'student'" :min="dateValidation.birthDate.min" :max="dateValidation.birthDate.max"/>
                  <span v-if="errors.dateOfBirth" class="error-message">{{ errors.dateOfBirth }}</span>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">民族</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.ethnicity || '未填写' }}</span>
                  <input v-else v-model="editableStudent.ethnicity" class="form-input" placeholder="请输入民族"/>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">籍贯</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.nativePlace || '未填写' }}</span>
                  <input v-else v-model="editableStudent.nativePlace" class="form-input" placeholder="请输入籍贯"/>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">政治面貌</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.politicalStatus || '未填写' }}</span>
                  <select v-else v-model="editableStudent.politicalStatus" class="form-select">
                    <option value="群众">群众</option>
                    <option value="共青团员">共青团员</option>
                    <option value="中共预备党员">中共预备党员</option>
                    <option value="中共党员">中共党员</option>
                    <option value="其他">其他</option>
                  </select>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="info-card">
          <div class="card-header">
            <h4>联系方式</h4>
          </div>
          <div class="card-content">
            <div class="form-grid">
              <div class="info-item">
                <label class="item-label">手机号码</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.phoneNumber || '未填写' }}</span>
                  <input v-else v-model="editableStudent.phoneNumber" class="form-input"
                         placeholder="请输入手机号码" type="tel"/>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">电子邮箱</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.email || '未填写' }}</span>
                  <input v-else type="email" v-model="editableStudent.email" class="form-input"
                         placeholder="请输入电子邮箱"/>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="info-card">
          <div class="card-header">
            <h4>学籍信息</h4>
          </div>
          <div class="card-content">
            <div class="form-grid">
              <div class="info-item">
                <label class="item-label">学号</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value primary-text">{{ student.studentId }}</span>
                  <input v-else v-model="editableStudent.studentId" class="form-input" :class="{'is-invalid': errors.studentId}" :disabled="!isCreatingNew" placeholder="请输入学号" />
                  <span v-if="errors.studentId" class="error-message">{{ errors.studentId }}</span>
                </div>
              </div>
              <div class="info-item">
                <label class="item-label">入学日期</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ formatDate(student.enrollmentDate) }}</span>
                  <input v-else type="date" v-model="editableStudent.enrollmentDate" class="form-input" :disabled="viewMode === 'student'" :min="dateValidation.enrollmentDate.min" :max="dateValidation.enrollmentDate.max"/>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">学院</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.college || '未填写' }}</span>
                  <input v-else v-model="editableStudent.college" class="form-input"
                         :disabled="viewMode === 'student'" placeholder="请输入学院"/>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">班级</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.className || '未分配' }}</span>
                  <input v-else v-model="editableStudent.className" class="form-input" :class="{'is-invalid': errors.className}" :disabled="viewMode === 'student'" placeholder="请输入班级"/>
                  <span v-if="errors.className" class="error-message">{{ errors.className }}</span>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">专业</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.majorName || '未分配' }}</span>
                  <template v-else>
                    <input v-if="viewMode === 'student'"
                           :value="editableStudent.majorName || '未分配'"
                           class="form-input"
                           disabled />
                    <select v-else-if="viewMode === 'admin'"
                            v-model="editableStudent.majorId"
                            class="form-select">
                      <option :value="null">未分配</option>
                      <option v-for="major in majors" :key="major.id" :value="major.id">
                        {{ major.name }}
                      </option>
                    </select>
                  </template>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">当前学年</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">
                    {{ student.academicYear ? `第 ${student.academicYear} 学年` : '未设置' }}
                  </span>
                  <select v-else v-model.number="editableStudent.academicYear" class="form-select" :disabled="viewMode === 'student'">
                    <option :value="null">未设置</option>
                    <option v-for="n in 4" :key="n" :value="n">第 {{ n }} 学年</option>
                  </select>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">当前学期</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ formatSemester(student.semester) }}</span>
                  <select v-else v-model.number="editableStudent.semester" class="form-select" :disabled="viewMode === 'student'">
                    <option :value="null">未设置</option>
                    <option value="1">上学期</option>
                    <option value="2">下学期</option>
                  </select>
                </div>
              </div>

              <div class="info-item">
                <label class="item-label">学籍状态</label>
                <div class="item-content">
                  <span v-if="!isEditing" class="item-value">{{ student.studentStatus }}</span>
                  <select v-else v-model="editableStudent.studentStatus" class="form-select" :disabled="viewMode === 'student'">
                    <option value="在读">在读</option>
                    <option value="休学">休学</option>
                    <option value="毕业">毕业</option>
                    <option value="退学">退学</option>
                  </select>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="info-card" v-if="isEditing">
          <div class="card-header">
            <h4>{{ viewMode === 'admin' ? '账户安全' : '修改密码' }}</h4>
          </div>
          <div class="card-content">
            <div class="form-grid">
              <div class="info-item full-width">
                <label class="item-label">
                  {{ isCreatingNew ? '设置初始密码' : (viewMode === 'admin' ? '重置密码' : '新密码') }}
                </label>
                <div class="item-content">
                  <input type="password" v-model="editableStudent.password" class="form-input" :class="{'is-invalid': errors.password}" :placeholder="getPasswordPlaceholder()"/>
                  <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
                  <p v-else class="field-hint">
                    {{ getPasswordHint() }}
                  </p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import { showNotification } from '@/services/notificationStore.js';
import { majorService } from '@/services/apiService';

const props = defineProps({
  student: { type: Object, required: true },
  viewMode: { type: String, default: 'admin' },
  isSaving: { type: Boolean, default: false }
});

const emit = defineEmits(['create-student', 'update-student', 'delete-student']);

const majors = ref([]);
const isEditing = ref(false);
const editableStudent = ref({});
const errors = ref({});

const isCreatingNew = computed(() => !props.student.id);

onMounted(async () => {
  if (props.viewMode === 'admin') {
    try {
      majors.value = await majorService.getAll();
    } catch (error) {
      showNotification('加载专业列表失败', 'error');
    }
  }
});

watch(() => props.student, (newStudent) => {
  if (newStudent) {
    editableStudent.value = JSON.parse(JSON.stringify(newStudent));
    editableStudent.value.password = '';
    isEditing.value = isCreatingNew.value;
    errors.value = {};
  }
}, { immediate: true, deep: true });

const getPasswordPlaceholder = () => isCreatingNew.value ? '请输入初始密码（必填）' : '不修改请留空';
const getPasswordHint = () => isCreatingNew.value ? '建议使用包含字母和数字的组合密码。' : (props.viewMode === 'admin' ? '输入新密码将重置该学生的登录密码。' : '输入新密码以更新您的登录密码。');
const formatDate = (d) => d ? new Date(d).toLocaleDateString('zh-CN', { year: 'numeric', month: 'long', day: 'numeric' }) : '未填写';
const formatSemester = (s) => s === 1 ? '上学期' : (s === 2 ? '下学期' : '未设置');
const getStatusClass = (s) => ({ '休学': 'status-warning', '毕业': 'status-success', '退学': 'status-danger' }[s] || '');

const startEditing = () => { isEditing.value = true; };

const cancelEditing = () => {
  editableStudent.value = JSON.parse(JSON.stringify(props.student));
  editableStudent.value.password = '';
  isEditing.value = false;
  errors.value = {};
};

const dateValidation = computed(() => {
  const today = new Date().toISOString().split('T')[0];
  const currentYear = new Date().getFullYear();
  return {
    birthDate: { min: '1920-01-01', max: today },
    enrollmentDate: { min: '2000-01-01', max: `${currentYear + 5}-12-31` }
  };
});

const validateForm = () => {
  const newErrors = {};
  const { name, studentId, gender, dateOfBirth, className, password } = editableStudent.value;

  if (!name) newErrors.name = '姓名不能为空';
  else if (name.length < 2 || name.length > 50) newErrors.name = '姓名长度必须在2到50之间';

  if (!studentId) newErrors.studentId = '学号不能为空';
  else if (studentId.length < 4 || studentId.length > 20) newErrors.studentId = '学号长度必须在4到20之间';

  if (!gender) newErrors.gender = '性别不能为空';
  if (!dateOfBirth) newErrors.dateOfBirth = '出生日期不能为空';
  if (!className) newErrors.className = '班级名称不能为空';

  if (isCreatingNew.value && !password) newErrors.password = '创建学生时必须设置初始密码';

  errors.value = newErrors;
  return Object.keys(newErrors).length === 0;
};

const handleSave = async () => {
  errors.value = {};

  if (!validateForm()) {
    showNotification('部分字段填写不正确，请检查。', 'error');
    return;
  }

  if (props.viewMode === 'admin' && editableStudent.value.majorId) {
    const selectedMajor = majors.value.find(m => m.id === editableStudent.value.majorId);
    if (selectedMajor) editableStudent.value.majorName = selectedMajor.name;
  }

  try {
    const eventName = isCreatingNew.value ? 'create-student' : 'update-student';
    // 注意：这里需要父组件来处理Promise的reject，所以我们直接emit
    emit(eventName, editableStudent.value);
  } catch (error) {
    // 这个catch块可能不会被直接触发，因为emit是同步的。
    // 错误处理应当在父组件的事件处理器中进行。
    // 为了保险起见，我们在这里也处理一下。
    if (error.status === 400 && error.data) {
      errors.value = error.data;
      showNotification(error.message || '提交的数据有误', 'error');
    } else {
      showNotification(error.message || '操作失败', 'error');
    }
  }
};

const handleDelete = () => {
  if (!isCreatingNew.value) {
    // 确认操作现在由父组件处理
    emit('delete-student', props.student.id);
  }
};
</script>

<style scoped>
.detail-container {
  height: 100%;
  background: linear-gradient(135deg, var(--color-background) 0%, #f8fafc 50%, var(--color-primary-light) 100%);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.page-header {
  background: linear-gradient(135deg, rgba(var(--color-primary-rgb), 0.95), rgba(var(--color-primary-rgb), 0.85));
  color: white;
  padding: var(--spacing-xl);
  box-shadow: 0 4px 20px rgba(var(--color-primary-rgb), 0.3);
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: var(--spacing-xl);
}

.header-info {
  flex: 1;
}

.page-title {
  font-size: 2.2em;
  font-weight: 700;
  margin: 0 0 var(--spacing-sm);
  letter-spacing: -0.5px;
}

.page-subtitle {
  font-size: 1.1em;
  margin: 0 0 var(--spacing-md);
  opacity: 0.9;
}

.status-badges {
  display: flex;
  gap: var(--spacing-sm);
  flex-wrap: wrap;
}

.status-badge, .info-badge {
  padding: var(--spacing-xs) var(--spacing-md);
  border-radius: 20px;
  font-size: 0.85em;
  font-weight: 600;
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.status-warning { background-color: var(--color-warning) !important; color: white !important; }
.status-success { background-color: var(--color-success) !important; color: white !important; }
.status-danger { background-color: var(--color-danger) !important; color: white !important; }

.header-actions {
  display: flex;
  gap: var(--spacing-md);
  flex-shrink: 0;
  flex-wrap: wrap;
}

.action-btn {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  padding: 12px 20px;
  border: none;
  border-radius: var(--border-radius);
  font-weight: 600;
  font-size: 0.9em;
  cursor: pointer;
  transition: all var(--transition-speed) ease;
  white-space: nowrap;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.edit-btn { background: rgba(255, 255, 255, 0.2); color: white; }
.delete-btn { background: rgba(220, 53, 69, 0.2); color: white; }
.save-btn { background: var(--color-success); color: white; }
.cancel-btn { background: rgba(108, 117, 125, 0.2); color: white; }
.action-btn:hover:not(:disabled) { opacity: 0.8; transform: translateY(-1px); }
.action-btn:disabled { opacity: 0.6; cursor: not-allowed; }

.loading-spinner {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top: 2px solid white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

.content-wrapper { flex: 1; overflow-y: auto; padding: var(--spacing-xl); }
.content-grid { display: grid; grid-template-columns: 1fr; gap: var(--spacing-xl); max-width: 1200px; margin: 0 auto; }
.info-card { background: var(--color-surface); border-radius: calc(var(--border-radius) * 1.5); box-shadow: 0 8px 32px rgba(0, 0, 0, 0.06), 0 0 0 1px rgba(var(--color-primary-rgb), 0.05); overflow: hidden; transition: all var(--transition-speed) ease; }
.info-card::before { content: ''; display: block; height: 3px; background: linear-gradient(90deg, var(--color-primary), rgba(var(--color-primary-rgb), 0.6)); }
.info-card:hover { transform: translateY(-2px); box-shadow: 0 12px 40px rgba(0, 0, 0, 0.1); }
.card-header { padding: var(--spacing-xl) var(--spacing-xl) var(--spacing-lg); background: linear-gradient(135deg, rgba(var(--color-primary-rgb), 0.02), transparent); border-bottom: 1px solid rgba(var(--color-primary-rgb), 0.1); }
.card-header h4 { font-size: 1.3em; font-weight: 700; color: var(--color-text-primary); margin: 0; }
.card-content { padding: var(--spacing-lg) var(--spacing-xl) var(--spacing-xl); }
.form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(320px, 1fr)); gap: var(--spacing-xl); }
.info-item { display: flex; flex-direction: column; gap: var(--spacing-sm); }
.info-item.full-width { grid-column: 1 / -1; }
.item-label { font-weight: 600; color: var(--color-text-primary); font-size: 0.9em; }
.item-value { padding: 12px 0; color: var(--color-text-primary); font-weight: 500; min-height: 1.5em; line-height: 1.5; }
.item-value.primary-text { color: var(--color-primary); font-weight: 700; font-size: 1.1em; }
.form-input, .form-select { width: 100%; padding: 14px 18px; border: 2px solid var(--color-border); border-radius: var(--border-radius); font-size: 1em; font-family: inherit; background: #fcfdff; transition: all var(--transition-speed) ease; box-sizing: border-box; }
.form-input:focus, .form-select:focus { outline: none; border-color: var(--color-primary); background: white; box-shadow: 0 0 0 3px rgba(var(--color-primary-rgb), 0.1); transform: translateY(-1px); }
.form-input:disabled, .form-select:disabled { background: #f8f9fa; cursor: not-allowed; color: var(--color-text-secondary); }
.form-select { -webkit-appearance: none; -moz-appearance: none; appearance: none; background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'%3e%3cpath fill='none' stroke='%23343a40' stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M2 5l6 6 6-6'/%3e%3c/svg%3e"); background-repeat: no-repeat; background-position: right 16px center; background-size: 16px 12px; padding-right: 50px; }
.field-hint { margin: var(--spacing-sm) 0 0; font-size: 0.85em; color: var(--color-text-secondary); line-height: 1.4; }

@media (max-width: 768px) {
  .page-header { padding: var(--spacing-lg); }
  .header-content { flex-direction: column; gap: var(--spacing-lg); }
  .page-title { font-size: 1.8em; }
  .header-actions { width: 100%; justify-content: flex-start; }
  .action-btn { flex: 1; justify-content: center; }
  .content-wrapper { padding: var(--spacing-lg); }
  .form-grid { grid-template-columns: 1fr; }
}
@media (max-width: 480px) {
  .page-header { padding: var(--spacing-md); }
  .page-title { font-size: 1.6em; }
  .header-actions { flex-direction: column; }
  .action-btn { width: 100%; }
  .content-wrapper { padding: var(--spacing-md); }
  .card-header, .card-content { padding-left: var(--spacing-md); padding-right: var(--spacing-md); }
}

@keyframes shake {
  10%, 90% { transform: translate3d(-1px, 0, 0); }
  20%, 80% { transform: translate3d(2px, 0, 0); }
  30%, 50%, 70% { transform: translate3d(-4px, 0, 0); }
  40%, 60% { transform: translate3d(4px, 0, 0); }
}

.form-input.is-invalid,
.form-select.is-invalid {
  border-color: var(--color-danger);
  box-shadow: 0 0 0 3px rgba(220, 53, 69, 0.15);
  animation: shake 0.6s cubic-bezier(.36,.07,.19,.97) both;
}

.error-message {
  color: var(--color-danger);
  font-size: 0.85em;
  font-weight: 500;
  margin-top: var(--spacing-sm);
}
.item-content {
  position: relative;
}

.info-item .item-value + .error-message {
  display: block;
}
</style>