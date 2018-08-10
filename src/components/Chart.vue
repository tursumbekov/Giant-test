<script>
import { Line } from 'vue-chartjs';

export default {
  extends: Line,
  props: ['chartData', 'options'],
  watch: {
    chartData: {
      handler(val) {
        if (val.datasets.length && val.datasets[0].data.length) {
          this.renderChart(this.chartData, this.options);
        }
      },
      immediate: true,
      deep: true,
    },
  },
};
</script>
