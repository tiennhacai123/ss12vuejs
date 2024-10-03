<template>
  <div class="main-container">
    <div class="header-section">
      <h1>Hệ thống quản lý sách</h1>

      <select v-model="activeFilter" @change="applyFilter" class="select-filter">
        <option value="">Tất cả</option>
        <option value="Đã trả">Đã trả</option>
        <option value="Chưa trả">Chưa trả</option>
      </select>

      <button @click="toggleForm" class="btn-add">Thêm sách</button>
    </div>

    <table class="table-books">
      <thead>
        <tr>
          <th>#</th>
          <th>Tên sách</th>
          <th>Sinh viên</th>
          <th>Ngày mượn</th>
          <th>Ngày trả</th>
          <th>Tình trạng</th>
          <th>Thao tác</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(book, idx) in filteredList" :key="idx">
          <td>{{ idx + 1 }}</td>
          <td>{{ book.name }}</td>
          <td>{{ book.student }}</td>
          <td>{{ book.borrowDate }}</td>
          <td>{{ book.returnDate }}</td>
          <td>
            <select v-model="book.status" @change="persistBooks">
              <option value="Đã trả">Đã trả</option>
              <option value="Chưa trả">Chưa trả</option>
            </select>
          </td>
          <td>
            <button @click="startEdit(idx)" class="btn-edit">Sửa</button>
            <button @click="confirmDelete(idx)" class="btn-delete">Xóa</button>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="form-overlay" v-if="isFormVisible">
      <form class="form-container" @submit.prevent="saveBook">
        <h4>{{ isUpdating ? 'Chỉnh sửa sách' : 'Thêm sách mới' }}</h4>
        <input v-model="bookForm.name" type="text" placeholder="Tên sách" required />
        <input v-model="bookForm.student" type="text" placeholder="Người mượn" required />
        <input v-model="bookForm.borrowDate" type="date" required />
        <input v-model="bookForm.returnDate" type="date" required />
        <button type="submit" class="btn-submit">Lưu</button>
        <button @click="cancelForm" type="button" class="btn-cancel">Hủy</button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const isFormVisible = ref(false);
const isUpdating = ref(false);
const bookList = ref([]);
const filteredList = ref([]);
const bookForm = ref({ 
  name: '',
  student: '',
  borrowDate: '', 
  returnDate: '', 
  status: 'Chưa trả' });
const editIndex = ref(null);
const activeFilter = ref('');

onMounted(() => {
  const storedBooks = localStorage.getItem('bookList');
  if (storedBooks) bookList.value = JSON.parse(storedBooks);
  applyFilter();
});

const toggleForm = () => { 
  isFormVisible.value = true; 
};
const cancelForm = () => { 
  isFormVisible.value = false; resetForm(); 
};
const saveBook = () => {
  if (isUpdating.value) {
    bookList.value.splice(editIndex.value, 1, { ...bookForm.value });
    isUpdating.value = false;
  } else {
    bookList.value.push({ ...bookForm.value });
  }
  persistBooks();
  cancelForm();
};
const startEdit = (index) => {
  isUpdating.value = true;
  bookForm.value = { ...bookList.value[index] };
  editIndex.value = index;
  isFormVisible.value = true;
};
const confirmDelete = (index) => {
  if (confirm("Bạn có chắc chắn muốn xóa sách này?")) {
    removeBook(index);
    alert("Sách đã được xóa");
  }
};
const removeBook = (index) => { 
  bookList.value.splice(index, 1); 
  persistBooks(); 
  applyFilter(); 
};
const resetForm = () => { 
  bookForm.value = { 
    name: '', 
    student: '', 
    borrowDate: '', 
    returnDate: '', 
    status: 'Chưa trả' };
};
const persistBooks = () => { 
  localStorage.setItem('bookList', JSON.stringify(bookList.value)); 
  applyFilter(); 
};
const applyFilter = () => { 
  filteredList.value = activeFilter.value ? bookList.value.filter(book => book.status === activeFilter.value) : bookList.value; 
};
</script>

<style scoped>
.main-container {
  width: 85%;
  margin: 0 auto;
  padding: 25px;
  background-color: #f5f7fa;
  border-radius: 10px;
}

.header-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
}

.select-filter {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #aaa;
}

.btn-add {
  background-color: #17a2b8;
  color: white;
  padding: 10px 25px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.table-books {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 25px;
}

.table-books th, .table-books td {
  border: 1px solid #ddd;
  padding: 15px;
  text-align: center;
}

.table-books th {
  background-color: #f1f1f1;
}

.btn-edit {
  background-color: #f0ad4e;
  color: white;
  padding: 7px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn-delete {
  background-color: #d9534f;
  color: white;
  padding: 7px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.form-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
}

.form-container {
  background: white;
  padding: 25px;
  border-radius: 10px;
  width: 450px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-container input {
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 5px;
  width: 100%;
}

.btn-submit {
  background-color: #007bff;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn-cancel {
  background-color: #6c757d;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>
