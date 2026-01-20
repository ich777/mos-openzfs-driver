<template>
  <div>
    <h2 class="mb-4">ZFS Driver</h2>
    <v-skeleton-loader v-if="loading" :loading="true" type="card" />
    <div v-else style="margin-bottom: 80px">
      <v-card class="mb-4 pa-0">
        <v-card-title>ZFS Pools</v-card-title>
        <v-card-text class="pa-4">
          <div v-if="pools.length === 0" class="text-center text-grey py-4">
            No ZFS pools found
          </div>
          <v-table v-else density="compact">
            <thead>
              <tr>
                <th>Name</th>
                <th>Size</th>
                <th>Used</th>
                <th>Free</th>
                <th>Health</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="pool in pools" :key="pool.name">
                <td class="font-weight-medium">{{ pool.name }}</td>
                <td>{{ pool.size }}</td>
                <td>{{ pool.allocated }}</td>
                <td>{{ pool.free }}</td>
                <td>
                  <v-chip :color="pool.health === 'ONLINE' ? 'success' : 'error'" size="small">
                    {{ pool.health }}
                  </v-chip>
                </td>
              </tr>
            </tbody>
          </v-table>
        </v-card-text>
      </v-card>

      <v-card v-if="driverInfo && driverInfo.package" class="mb-4 pa-0">
        <v-card-title>Installed Package</v-card-title>
        <v-card-text class="pa-4">
          <v-row dense>
            <v-col cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Plugin</strong></div>
              <div class="text-body-2">{{ driverInfo.plugin || '-' }}</div>
            </v-col>
            <v-col cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Kernel</strong></div>
              <div class="text-body-2">{{ driverInfo.kernel || '-' }}</div>
            </v-col>
            <v-col cols="12" md="6">
              <div class="text-caption text-medium-emphasis"><strong>Package</strong></div>
              <div class="text-body-2" style="word-break: break-all">{{ driverInfo.package || '-' }}</div>
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const PLUGIN_NAME = 'zfs-driver';

const loading = ref(true);
const pools = ref([]);
const driverInfo = ref(null);

const getAuthHeaders = () => ({
  Authorization: 'Bearer ' + localStorage.getItem('authToken'),
});

const fetchPools = async () => {
  try {
    const res = await fetch('/api/v1/mos/plugins/query', {
      method: 'POST',
      headers: {
        ...getAuthHeaders(),
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        command: 'get_zfs_pools',
        args: [],
        timeout: 10,
        parse_json: true,
      }),
    });

    if (res.ok) {
      const data = await res.json();
      const output = data.output || {};
      const poolsObj = output.pools || {};
      pools.value = Object.values(poolsObj).map((p) => ({
        name: p.name,
        size: p.properties?.size?.value || '-',
        allocated: p.properties?.allocated?.value || '-',
        free: p.properties?.free?.value || '-',
        health: p.properties?.health?.value || 'UNKNOWN',
      }));
    }
  } catch (e) {
    console.error('Failed to fetch ZFS pools:', e);
  }
};

const fetchDriverInfo = async () => {
  try {
    const res = await fetch(`/api/v1/mos/plugins/driver/${PLUGIN_NAME}`, {
      headers: getAuthHeaders(),
    });
    if (res.ok) {
      driverInfo.value = await res.json();
    }
  } catch (e) {
    console.error('Failed to fetch driver info:', e);
  }
};

onMounted(async () => {
  try {
    await Promise.all([fetchPools(), fetchDriverInfo()]);
  } catch (e) {
    console.error('Failed to initialize:', e);
  } finally {
    loading.value = false;
  }
});
</script>
