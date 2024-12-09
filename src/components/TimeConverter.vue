<template>
  <div>
    <h1>Time Converter</h1>
    <div class="action-buttons">
      <input
        type="file"
        ref="fileInput"
        style="display: none"
        accept=".json"
        @change="handleFileUpload"
      />
      <button class="action-btn" @click="$refs.fileInput.click()">
        Import Config
      </button>
      <button class="action-btn" @click="downloadConfig">
        Export Config
      </button>
      <button class="action-btn save-btn" @click="saveToLocal">
        Save to Browser
      </button>
    </div>
    <div v-for="(config, index) in configs" :key="index">
      <div class="config-row">
        <button 
          @click="removeConfig(index)" 
          class="delete-btn"
          v-if="configs.length > 1"
        >
          ×
        </button>
        <input v-model="config.name" placeholder="Name" />
        <div class="select-container">
          <input 
            v-model="config.timezone" 
            placeholder="Timezone" 
            @focus="showTimezoneDropdown(index)"
          />
          <div class="dropdown" v-if="activeDropdown === `timezone-${index}`">
            <div 
              v-for="tz in commonTimezones" 
              :key="tz"
              class="dropdown-item"
              @click="selectTimezone(index, tz)"
            >
              {{ tz }}
            </div>
          </div>
        </div>
        <div class="select-container">
          <input 
            v-model="config.format" 
            placeholder="Format"
            @focus="showFormatDropdown(index)"
          />
          <div class="dropdown" v-if="activeDropdown === `format-${index}`">
            <div 
              v-for="fmt in predefinedFormats" 
              :key="fmt.value"
              class="dropdown-item"
              @click="selectFormat(index, fmt.value)"
            >
              {{ fmt.label }}
            </div>
          </div>
        </div>
        <input 
          v-model="config.inputTime" 
          @keyup.enter="convertTime(index)" 
          @focus="activeIndex = index"
          :class="{ active: activeIndex === index, inactive: activeIndex !== null && activeIndex !== index }"
          placeholder="Enter time" 
        />
      </div>
    </div>
    <button @click="addConfig" class="add-config-btn">Add Configuration</button>
  </div>
</template>

<script>
import moment from 'moment-timezone';

export default {
  data() {
    return {
      configs: [
        { name: '', timezone: 'UTC', format: 'Timestamp', inputTime: '', convertedTime: '' }
      ],
      activeIndex: null,
      activeDropdown: null,
      commonTimezones: [
        'UTC',
        'UTC+1',
        'UTC+2',
        'UTC+3',
        'UTC+4',
        'UTC+5',
        'UTC+6',
        'UTC+7',
        'UTC+8',
        'UTC+9',
        'UTC+10',
        'UTC+11',
        'UTC+12',
        'UTC-1',
        'UTC-2',
        'UTC-3',
        'UTC-4',
        'UTC-5',
        'UTC-6',
        'UTC-7',
        'UTC-8',
        'UTC-9',
        'UTC-10',
        'UTC-11',
        'UTC-12'
      ],
      predefinedFormats: [
        { label: 'Timestamp', value: 'Timestamp' },
        { label: 'yyyy-MM-DD HH:mm:ss', value: 'YYYY-MM-DD HH:mm:ss' },
        { label: 'yyyy-MM-DD HH:mm', value: 'YYYY-MM-DD HH:mm' },
        { label: 'yyyy-MM-DD', value: 'YYYY-MM-DD' },
        { label: 'MM/DD/yyyy HH:mm:ss', value: 'MM/DD/YYYY HH:mm:ss' },
        { label: 'MM/DD/yyyy HH:mm', value: 'MM/DD/YYYY HH:mm' },
        { label: 'MM/DD/yyyy', value: 'MM/DD/YYYY' },
        { label: 'DD/MM/yyyy HH:mm:ss', value: 'DD/MM/YYYY HH:mm:ss' },
        { label: 'DD/MM/yyyy HH:mm', value: 'DD/MM/YYYY HH:mm' },
        { label: 'DD/MM/yyyy', value: 'DD/MM/YYYY' },
        { label: 'HH:mm:ss', value: 'HH:mm:ss' },
        { label: 'HH:mm', value: 'HH:mm' }
      ]
    };
  },
  methods: {
    addConfig() {
      this.configs.push({ name: '', timezone: 'UTC', format: 'Timestamp', inputTime: '', convertedTime: '' });
    },
    showTimezoneDropdown(index) {
      this.activeDropdown = `timezone-${index}`;
    },
    showFormatDropdown(index) {
      this.activeDropdown = `format-${index}`;
    },
    selectTimezone(index, timezone) {
      this.configs[index].timezone = timezone;
      this.activeDropdown = null;
    },
    selectFormat(index, format) {
      this.configs[index].format = format;
      this.activeDropdown = null;
    },
    removeConfig(index) {
      this.configs.splice(index, 1);
      if (this.activeIndex === index) {
        this.activeIndex = null;
      } else if (this.activeIndex > index) {
        this.activeIndex--;
      }
    },
    convertTime(index) {
      const inputConfig = this.configs[index];
      const inputFormat = inputConfig.format === 'Timestamp' ? '' : inputConfig.format;
      let inputTime;

      // 尝试解析时间戳或格式化的时间字符串
      if (inputConfig.format === 'Timestamp') {
        // 如果没有指定格式，则尝试解析时间戳
        const timestamp = parseInt(inputConfig.inputTime);
        if (isNaN(timestamp)) {
          alert('Invalid timestamp');
          return;
        }
        inputTime = moment(timestamp);
      } else {
        // 如果指定了格式，则按照格式解析
        const parsedTime = moment(inputConfig.inputTime, inputFormat, true);
        if (!parsedTime.isValid()) {
          alert(`Invalid time format. Please enter time in ${inputFormat} format`);
          return;
        }
        inputTime = parsedTime;
      }

      // 解析时区偏移
      let timezone = inputConfig.timezone;
      if (timezone.startsWith('UTC')) {
        const offset = timezone.substring(3);
        if (offset) {
          timezone = `Etc/GMT${offset.startsWith('+') ? '-' : '+'}${offset.substring(1)}`;
        }
      }

      // 将时间转换到指定时区
      inputTime = inputTime.tz(timezone);

      // 将输入时间转换为UTC时间戳
      const utcTimestamp = inputTime.valueOf();

      // 更新所有配置项的输入框为转换结果
      this.configs.forEach((config, idx) => {
        if (idx !== index) {
          let targetTimezone = config.timezone;
          if (targetTimezone.startsWith('UTC')) {
            const offset = targetTimezone.substring(3);
            if (offset) {
              targetTimezone = `Etc/GMT${offset.startsWith('+') ? '-' : '+'}${offset.substring(1)}`;
            }
          }

          if (config.format === 'Timestamp') {
            // 如果没有指定格式，输出时间戳
            config.inputTime = moment(utcTimestamp).tz(targetTimezone).valueOf().toString();
          } else {
            // 如果指定了格式，按格式输出
            config.inputTime = moment(utcTimestamp).tz(targetTimezone).format(config.format);
          }
        }
      });

      // 更新当前配置项的输入框为转换结果
      if (inputConfig.format === 'Timestamp') {
        inputConfig.inputTime = inputTime.valueOf().toString();
      } else {
        inputConfig.inputTime = inputTime.format(inputFormat);
      }
    },
    downloadConfig() {
      const configData = JSON.stringify(this.configs, null, 2);
      const blob = new Blob([configData], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'time-converter-config.json';
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          try {
            const configs = JSON.parse(e.target.result);
            if (Array.isArray(configs) && configs.every(config => 
              typeof config === 'object' && 
              'name' in config && 
              'timezone' in config && 
              'format' in config && 
              'inputTime' in config
            )) {
              this.configs = configs;
            } else {
              alert('Invalid configuration file format');
            }
          } catch (error) {
            alert('Error reading configuration file');
          }
        };
        reader.readAsText(file);
      }
      // 清除文件输入，允许重复上传同一个文件
      event.target.value = '';
    },
    saveToLocal() {
      try {
        localStorage.setItem('timeConverterConfigs', JSON.stringify(this.configs));
        alert('Configuration saved successfully');
      } catch (error) {
        alert('Failed to save configuration');
      }
    },
    loadFromLocal() {
      const savedConfigs = localStorage.getItem('timeConverterConfigs');
      if (savedConfigs) {
        try {
          const configs = JSON.parse(savedConfigs);
          if (Array.isArray(configs) && configs.every(config => 
            typeof config === 'object' && 
            'name' in config && 
            'timezone' in config && 
            'format' in config && 
            'inputTime' in config
          )) {
            this.configs = configs;
          }
        } catch (error) {
          console.error('Error loading saved configuration:', error);
        }
      }
    }
  },
  mounted() {
    // 点击页面其他地方时关闭下拉框
    document.addEventListener('click', (e) => {
      if (!e.target.closest('.select-container')) {
        this.activeDropdown = null;
      }
    });
    // 加载保存的配置
    this.loadFromLocal();
  }
};
</script>

<style scoped>
.config-row {
  display: flex;
  align-items: center;
  margin-bottom: 8px;
  padding: 0;
}

input {
  transition: opacity 0.3s;
  margin-right: 8px;
}

.active {
  opacity: 1;
  font-weight: bold;
}

.inactive {
  opacity: 0.5;
}

.delete-btn {
  width: 20px;
  height: 20px;
  line-height: 16px;
  font-size: 16px;
  color: #fff;
  background-color: #ff4d4f;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  margin-right: 8px;
}

.delete-btn:hover {
  background-color: #ff7875;
}

.select-container {
  position: relative;
  margin-right: 8px;
}

.dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  max-height: 200px;
  overflow-y: auto;
  z-index: 1000;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
}

.dropdown-item {
  padding: 8px 12px;
  cursor: pointer;
}

.dropdown-item:hover {
  background-color: #f5f5f5;
}

/* 名称输入框 */
.config-row > input:first-of-type {
  width: 150px;
}

/* 时区输入框 */
.config-row .select-container:first-of-type input {
  width: 75px;
}

/* 格式输入框 */
.config-row .select-container:nth-of-type(2) input {
  width: 200px;
}

/* 时间输入框 */
.config-row > input:last-of-type {
  width: 150px;
}

.action-buttons {
  margin-bottom: 16px;
  display: flex;
  gap: 8px;
}

.action-btn {
  padding: 4px 8px;
  background-color: #1890ff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

.action-btn:hover {
  background-color: #40a9ff;
}

.add-config-btn {
  padding: 4px 8px;
  background-color: #52c41a;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  margin-top: 8px;
}

.add-config-btn:hover {
  background-color: #73d13d;
}

.save-btn {
  background-color: #722ed1;
}

.save-btn:hover {
  background-color: #9254de;
}
</style> 