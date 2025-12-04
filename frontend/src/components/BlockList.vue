<template>
    <div class="blockchain-container">
        <div class="header">
            <h1>Blockchain Visualization</h1>
            <div class="header-right">
                <div class="node-info">
                    <span class="label">当前节点：</span>
                    <span class="current-node">P2P未知</span>
                </div>
                <div class="status">
                    <span :class="['status-dot', connected ? 'online' : 'offline']"></span>
                    {{ connected ? '已连接' : '未连接' }}
                </div>
            </div>
        </div>
       
        <div class="node-selector-bar">
            <div class="selector-left">
                <span class="label">可用节点：</span>
                <button
                    v-for="node in availableNodes"
                    :key="node.airPort"
                    @click="switchToNode(node)"
                    :class="['node-btn', { active: currentApiPort === node.airPort }]"
                    :title="`api端口：${node.airPort}, 区块数：${node.blockCount}`"
                >
                    <div class="node-id">节点：</div>
                    <div class="node-blocks">区块</div>
                </button>
                <div v-if="availableNodes.length === 0 && !isDiscovering" class="no-nodes">未发现节点</div>
                <div v-if="isDiscovering" class="discovering">扫描中....</div>
            </div>
            <button @click="discoverNodes" class="refresh-nodes-btn" :disabled="isDiscovering">刷新节点</button>
        </div>
         <div class="controls">
            <button @click="fetchBlocks" class="refresh-btn">
                Refresh Chain
            </button>
            <div class="auto-refresh">
                <label>
                    <input type="checkbox" v-model="autoRefresh"></input>
                    Auto Refresh(2s)
                </label>
            </div>
        </div>

        <div class="chain-display" v-if="blocks.length > 0">
            <div v-for="(block, index) in blocks" :key="block.index" class="block-wrapper">
                <div class="block-card">
                    <div class="block-header">
                        <span class="block-index">#{{ block.index }}</span>
                        <span class="block-time">{{ block.timestamp }}</span>
                    </div>
                    <div class="block-body">
                        <div class="data-row">
                            <span class="label">Hash:</span>
                            <span class="value hash" :title="block.hash">{{ block.hash }}</span>
                        </div>
                        <div class="data-row">
                            <span class="label">Prev Hash:</span>
                            <span class="value hash">{{ block.previousHash }}</span>
                        </div>
                        <div class="data-row">
                            <span class="label">Data:</span>
                            <span class="value data-content">{{ block.data }}</span>
                        </div>
                        <div class="data-row">
                            <span class="label">Nonce:</span>
                            <span class="value">{{ block.nonce }}</span>
                        </div>
                    </div>
                </div>
                <div v-if="index < blocks.length - 1" class="chain-link">
                    <div class="arrow">↓</div>
                </div>
            </div>
        </div>
        <div v-else class="empty-state">
            Waiting for blockchain data...
        </div>
    </div>
   
</template>

<script setup>
import { ref } from 'vue';
const connected = ref(false);
const availableNodes = ref([]);
const currentApiPort = ref(3001);
const blocks = ref([
    {
        index: 0,
        previousHash: "0",
        timestamp: 1508270000000,
        data: "first block",
        hash: "000dc75a315c77a1f9c98fb6247d03dd18ac52632d7dc6a9920261d8109b37cf",
        nonce: 604
    },
    {
        index: 1,
        previousHash: "000dc75a315c77a1f9c98fb6247d03dd18ac52632d7dc6a9920261d8109b37cf",
        timestamp: 1733244720000,
        data: "Hello Blockchain!",
        hash: "000a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a",
        nonce: 1234
    },
    {
        index: 2,
        previousHash: "000a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a",
        timestamp: 1733244820000,
        data: "Second block data",
        hash: "000b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b",
        nonce: 5678
    },
    {
        index: 3,
        previousHash: "000b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b",
        timestamp: 1733244920000,
        data: "Transaction data: Alice -> Bob: 100 BTC",
        hash: "000c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c",
        nonce: 9012
    }
]);
const autoRefresh = ref(true);
const isDiscovering = ref(false);
let timer = null;
</script>
<style scoped>
.blockchain-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    font-family: 'Inter', sans-serif;
}
.header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    border-bottom: 1px solid #eee;
    padding-bottom: 20px;
}
.header-right {
    display: flex;
    align-items: center;
    gap: 20px;
}
h1 {
    margin: 0;
    color: #2c3e50;
    font-size: 24px;
}
.node-info {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 6px 12px;
    background: #f8f9fa;
    border-radius: 6px;
}
.node-info .label {
    font-size: 13px;
    color: #666;
}
.current-node {
    font-weight: 600;
    color: #667eea;
    font-family: 'Roboto', sans-serif;
}
.status {
    display: flex;
    align-items: center;
    font-size: 14px;
    color: #666;
}
.status-dot {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    margin-right: 8px;
    background-color: #f41616;
}
.status-dot.online {
    background-color: #42b983;
    box-shadow: 0 0 8px rgba(66, 185, 131, 0.5);
}

.status-dot.offine {
    background-color: #f7422e;
}
.node-selector-bar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    padding: 15px;
    background: white;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    border: 1px solid #e8e8e8;
    gap: 15px;
    flex-wrap: wrap;
}
.selector-left {
    display: flex;
    align-items: center;
    gap: 12px;
    flex: 1;
    flex-wrap: wrap;
}
.selector-left .label {
    font-size: 14px;
    color: #666;
    font-weight: 500;
}
.node-btn {
    padding: 8px 14px;
    border: 2px solid #e0e0e0;
    background: white;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2px;
}
.node-btn:hover {
    border-color: #667eea;
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.2);
}
.node-btn.active {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-color: #667eea;
    color: white;
    box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
}
.node-id {
    font-size: 13px;
    font-weight: 600;
}
.node-blocks {
    font-size: 11px;
    opacity: 0.8;
}
.no-nodes, .discovering {
    font-size: 13px;
    color: #999;
    font-style: italic;
    padding: 8px 12px;
}
.discovering {
    color: #667eea;
}
.refresh-nodes-btn {
    padding: 8px 16px;
    background: #667eea;
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 13px;
    transition: all 0.3s ease;
    white-space: nowrap;
}
.refresh-nodes-btn:hover:not(:disabled) {
    background: #5568d3;
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}
.refresh-nodes-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}
.refresh-nodes-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}
.controls {
    margin-bottom: 30px;
    display: flex;
    align-items: center;
    gap: 20px;
}
.refresh-btn {
    background-color: #3498db;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 6px;
    cursor: pointer;
    transition: background-color 0.3s;
}
.refresh-btn:hover {
    background-color: #2980b9;
}
.chain-display {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.block-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
}

.block-card {
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 600px;
    overflow: hidden;
    transition: transform 0.3s;
    border: 1px solid #f0f0f0;
}

.block-card:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
}

.block-header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 12px 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.block-index {
    font-weight: bold;
    font-size: 18px;
}

.block-time {
    font-size: 12px;
    opacity: 0.9;
}

.block-body {
    padding: 20px;
}

.data-row {
    display: flex;
    margin-bottom: 10px;
    font-size: 14px;
    line-height: 1.5;
}

.data-row:last-child {
    margin-bottom: 0;
}

.label {
    width: 100px;
    color: #888;
    flex-shrink: 0;
}
.value {
  color: #333;
  word-break: break-all;
}

.data-content {
  font-weight: bold;
  color: #2c3e50;
}

.hash {
  color: #e67e22;
}

.chain-link {
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #ccc;
  font-size: 24px;
}

.empty-state {
  text-align: center;
  padding: 40px;
  color: #999;
  font-style: italic;
}
</style>