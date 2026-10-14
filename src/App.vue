<script setup>
import { ref, computed, onMounted } from "vue";
import axios from "axios";

const API = "http://localhost:3000/asset/";
const assets = ref([]);
const editId = ref(null);
const form = ref({
  asset_name: "",
  stock_quantity: 0,
  category: "Consumable",
});

const load = async () => {
  try {
    assets.value = (await axios.get(API)).data;
  } catch (e) {
    alert("Gagal mengambil data aset!");
  }
};

const reset = () => {
  editId.value = null;
  form.value = { asset_name: "", stock_quantity: 0, category: "Consumable" };
};

const save = async () => {
  const data = {
    ...form.value,
    stock_quantity: Number(form.value.stock_quantity),
  };

  try {
    if (editId.value !== null) await axios.put(API + editId.value, data);
    else await axios.post(API, data);

    reset();
    await load();
  } catch (e) {
    alert("Gagal menyimpan data!");
  }
};

const edit = (a) => {
  editId.value = a.asset_id;
  form.value = { ...a };
};

const remove = async (id) => {
  if (confirm("Hapus aset ini?")) {
    try {
      await axios.delete(API + id);
      await load();
    } catch (e) {
      alert("Gagal menghapus aset!");
    }
  }
};

onMounted(load);
</script>

<template>
  <main class="container">
    <header>
      <div>
        <h1>Asset Management</h1>
        <p>Kelola data inventaris dengan mudah.</p>
      </div>
      <button class="refresh" @click="load">↻ Refresh</button>
    </header>

    <section class="stats">
      <article>
        <span>Total Jenis Aset</span>
        <h2>{{ assets.length }}</h2>
      </article>

      <article>
        <span>Consumable</span>
        <h2>{{ assets.filter((a) => a.category === "Consumable").length }}</h2>
      </article>
    </section>

    <section class="panel">
      <h3>{{ editId !== null ? "Edit Aset" : "Tambah Aset" }}</h3>

      <form @submit.prevent="save">
        <label>Nama Aset</label>
        <input v-model="form.asset_name" placeholder="Nama aset..." required />

        <label>Jumlah Stok</label>
        <input
          v-model.number="form.stock_quantity"
          type="number"
          min="0"
          step="1"
          required
        />

        <label>Kategori</label>
        <select v-model="form.category">
          <option>Consumable</option>
          <option>Non-Consumable</option>
        </select>

        <div class="actions">
          <button
            v-if="editId !== null"
            type="button"
            class="cancel"
            @click="reset"
          >
            Batal
          </button>
          <button class="primary">
            {{ editId !== null ? "Simpan Perubahan" : "+ Tambah Aset" }}
          </button>
        </div>
      </form>
    </section>

    <section class="panel">
      <h3>
        Daftar Aset <span class="count">{{ assets.length }} aset</span>
      </h3>

      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>Nama Aset</th>
              <th>Stok</th>
              <th>Kategori</th>
              <th>Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="a in assets" :key="a.asset_id">
              <td>{{ a.asset_name }}</td>
              <td>{{ a.stock_quantity }}</td>
              <td>
                <span
                  class="badge"
                  :class="a.category === 'Consumable' ? 'orange' : 'blue'"
                >
                  {{ a.category }}
                </span>
              </td>
              <td>
                <button class="edit" @click="edit(a)">Edit</button>
                <button class="delete" @click="remove(a.asset_id)">
                  Hapus
                </button>
              </td>
            </tr>
            <tr v-if="assets.length === 0">
              <td colspan="4" class="empty">Belum ada data aset.</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
  </main>
</template>

<style>
* {
  box-sizing: border-box;
}
body {
  margin: 0;
  background: #f5f7fb;
  font:
    14px Arial,
    sans-serif;
  color: #263047;
}
.container {
  max-width: 1100px;
  margin: 35px auto;
  padding: 0 20px;
}
header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
}
h1 {
  margin: 0;
  font-size: 28px;
}
p {
  color: #8790a3;
}
.stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-bottom: 20px;
}
.stats article,
.panel {
  background: white;
  padding: 22px;
  border: 1px solid #e8ebf2;
  border-radius: 12px;
}
.stats span {
  color: #8790a3;
  font-size: 13px;
}
.stats h2 {
  margin: 12px 0 0;
  color: #6558e8;
  font-size: 27px;
}
.panel {
  margin-bottom: 20px;
}
h3 {
  display: flex;
  justify-content: space-between;
  margin: 0 0 20px;
}
label {
  display: block;
  margin: 15px 0 7px;
  font-weight: bold;
  font-size: 13px;
}
input,
select {
  width: 100%;
  padding: 11px;
  border: 1px solid #dfe3ed;
  border-radius: 7px;
  outline-color: #6558e8;
}
button {
  padding: 9px 12px;
  border: 0;
  border-radius: 7px;
  cursor: pointer;
  font-weight: bold;
}
.primary,
.refresh {
  background: #6558e8;
  color: white;
}
.actions {
  display: flex;
  gap: 8px;
  margin-top: 18px;
}
.cancel {
  background: #edf0f6;
}
.count {
  background: #f0efff;
  color: #6558e8;
  padding: 6px 10px;
  border-radius: 6px;
  font-size: 12px;
}
.table-wrap {
  overflow-x: auto;
}
table {
  width: 100%;
  border-collapse: collapse;
  white-space: nowrap;
}
th,
td {
  text-align: left;
  padding: 14px 10px;
  border-bottom: 1px solid #edf0f5;
}
th {
  background: #f8f9fc;
  color: #7f8799;
  font-size: 12px;
}
.badge {
  padding: 6px 9px;
  border-radius: 6px;
  font-size: 11px;
}
.orange {
  background: #fff1d9;
  color: #ad771c;
}
.blue {
  background: #e7f2ff;
  color: #347cc4;
}
.edit {
  background: #eeecff;
  color: #6558e8;
  margin-right: 5px;
}
.delete {
  background: #fff0f0;
  color: #d94e59;
}
.empty {
  text-align: center;
  color: #9299aa;
  padding: 25px;
}
@media (max-width: 600px) {
  .container {
    margin-top: 20px;
    padding: 0 12px;
  }
  h1 {
    font-size: 22px;
  }
  .stats {
    grid-template-columns: 1fr;
    gap: 10px;
  }
  .stats article {
    padding: 16px;
  }
  .panel {
    padding: 15px;
  }
}
</style>
```
