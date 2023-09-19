<template>
  <div class="container">

    <div class="checkpoints-container">
    <div v-for="(perspective, index) in perspectives" :key="index">
      <h2>{{ perspective.name }}</h2>
      <div v-for="(level, levelIndex) in perspective.levels" :key="levelIndex">
        <h3>Level {{ levelIndex + 1 }}</h3>
        <div>
          <label v-for="(checkpoint, checkpointIndex) in level.checkpoints" :key="checkpointIndex">
            <input type="checkbox" v-model="checkpoint.checked" @change="updateLevelCompletion(perspective, level)">
              <span style="color: black">{{ checkpoint.text }}</span>
          </label>
        </div>
      </div>
    </div>

    <div>
      <h2>Result</h2>
      <p v-if="isAllLevelsCompleted">All levels are completed!</p>
      <p v-else>Some levels are not completed.</p>
    </div>

  </div>

  <div class="chart-container">
    <svg :width="chartWidth" :height="chartHeight">
      <g :transform="`translate(${chartCenterX}, ${chartCenterY})`">
        <!-- Draw axes -->
        <g v-for="(axis, index) in chartData.labels" :key="index">
          <line
            :x1="0"
            :y1="0"
            :x2="getCoordinate(chartRadius, index, chartData.labels.length)[0]"
            :y2="getCoordinate(chartRadius, index, chartData.labels.length)[1]"
            stroke="lightgray"
          />
          <text
            :x="getCoordinate(chartRadius + labelOffset, index, chartData.labels.length)[0]"
            :y="getCoordinate(chartRadius + labelOffset, index, chartData.labels.length)[1]"
            dominant-baseline="middle"
            text-anchor="middle"
          >
            {{ axis }}
          </text>
        </g>

        <!-- Draw data points -->
        <g>
          <polygon
            :points="getPolygonPoints(chartData.data)"
            fill="rgba(75, 192, 192, 0.2)"
            stroke="rgba(75, 192, 192, 1)"
          />
          <polygon
            :points="getPolygonPoints(chartData.data2)"
            fill="rgba(192, 75, 192, 0.2)"
            stroke="rgba(192, 75, 192, 1)"
          />
          <circle
            v-for="(dataPoint, index) in chartData.data"
            :key="index"
            :cx="getCoordinate(chartRadius * (dataPoint / maxDataValue), index, chartData.data.length)[0]"
            :cy="getCoordinate(chartRadius * (dataPoint / maxDataValue), index, chartData.data.length)[1]"
            :r="pointRadius"
            fill="rgba(75, 192, 192, 1)"
          />
          <circle
            v-for="(dataPoint, index) in chartData.data2"
            :key="index"
            :cx="getCoordinate(chartRadius * (dataPoint / maxDataValue), index, chartData.data2.length)[0]"
            :cy="getCoordinate(chartRadius * (dataPoint / maxDataValue), index, chartData.data2.length)[1]"
            :r="pointRadius"
            fill="rgba(192, 75, 192, 1)"
          />
        </g>
      </g>
    </svg>
  </div>

  </div>
</template>

<script>


export default {

  data() {
    return {
       isAllLevelsCompleted: false,
        perspectives: [
        {
          name: '기능분해',
          levels: [
            {
              checkpoints: [
                { text: '비즈니스 역량 도출', checked: false },
                { text: '기능 정의', checked: false },
                { text: '유즈케이스 식별', checked: false },
                { text: '의존성 분석', checked: false },
              ],
              isCompleted: false,
            },
            {
              checkpoints: [
                { text: '인터페이스 설계', checked: false },
                { text: '데이터 액세스 계획', checked: false },
                { text: '보안 및 인증 고려', checked: false },
                { text: '에러 핸들링 정의', checked: false },
              ],
              isCompleted: false,
            },
            {
              checkpoints: [
                { text: 'Ubiquitous language 정의', checked: false },
                { text: 'Boundary 정의', checked: false },
                { text: 'Anti-corruption layer 구현', checked: false },
                { text: '도메인 이벤트 식별', checked: false },
              ],
              isCompleted: false,
            },
            {
              checkpoints: [
                { text: 'CQRS 모델 설계', checked: false },
                { text: '이벤트 스토리지 구현', checked: false },
                { text: '이벤트 드리븐 아키텍처 구축', checked: false },
                { text: '이벤트 버전 관리',checked: false },
              ],
              isCompleted: false,
            },
          ],
        },
        {
          name: '데이터',
          levels: [
            {
              checkpoints: [
                { text: '트랜잭션 관리', checked: false },
                { text: '데이터 일관성 유지', checked: false },
              ],
              isCompleted: false,
            },
            {
              checkpoints: [
                { text: '개별 스키마 정의', checked: false },
                { text: '다중 엔터프라이즈 데이터 조정', checked: false },
                { text: '데이터 일관성 유지', checked: false },
              ],
              isCompleted: false,
            },
            {
              checkpoints: [
                { text: '분산 데이터 관리', checked: false },
                { text: '폴리글랏 퍼시스턴스', checked: false },
                { text: '데이터 접근 및 관리', checked: false },
              ],
              isCompleted: false,
            },
            {
              checkpoints: [
                { text: '이벤트 기반 데이터 관리', checked: false },
                { text: '이벤트 소싱 및 커맨드-쿼리 분리(CQRS)', checked: false },
                { text: '데이터 일관성 관리', checked: false },
              ],
              isCompleted: false,
            },
          ],
        },
        // Add more perspectives here if needed
      ],

      chartData: {
        labels: ['기능분해', '데이터', 'SW 아키텍처', 'Infra', '스트럭처', '배포', '팀구조'],
        data: [0, 0, 0, 0, 0, 0, 0],
        data2: [0, 0, 0, 0, 0, 0, 0],
      },
      chartWidth: 400,
      chartHeight: 400,
      chartCenterX: 200,
      chartCenterY: 200,
      chartRadius: 150,
      labelOffset: 20,
      maxDataValue: 5,
      pointRadius: 4,
    };
  },
  methods: {
    getCoordinate(radius, index, total) {
      const angle = (Math.PI * 2 * index) / total - Math.PI / 2;
      const x = radius * Math.cos(angle);
      const y = radius * Math.sin(angle);
      return [x, y];
    },
    getPolygonPoints(data) {
      return data
        .map((dataPoint, index) => {
          const radius = this.chartRadius * (dataPoint / this.maxDataValue);
          return this.getCoordinate(radius, index, data.length).join(',');
        })
        .join(' ');
    },
    updateLevelCompletion(perspective, level) {
      level.isCompleted = level.checkpoints.every(checkpoint => checkpoint.checked);
      this.checkAllLevelsCompletion(perspective);
    },
 checkAllLevelsCompletion(perspective) {
  perspective.isCompleted = perspective.levels.every((level) => level.isCompleted);

//  if (perspective.isCompleted) {
    const perspectiveIndex = this.chartData.labels.findIndex((label) => label === perspective.name);
    if (perspectiveIndex !== -1) {
      const lastCompletedLevelIndex = perspective.levels.findIndex((level) => !level.isCompleted) - 1;
      const value = lastCompletedLevelIndex !== -1 ? lastCompletedLevelIndex + 1 : this.chartData.labels.length;
      this.chartData.data[perspectiveIndex] = value;
      this.chartData.data2[perspectiveIndex] = value;
    }
  //}
},
  }
};
</script>

<style>

.container {
  display: flex;
  justify-content: space-between;
}

.checkpoints-container {
  flex: 1;
}

.chart-container {
  flex: 1;
}
svg {
  display: block;
  margin: auto;
}
</style>

