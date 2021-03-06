### 自定义数据源

```java
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction;

import java.util.Calendar;
import java.util.Random;

public class SensorSource extends RichParallelSourceFunction<SensorReading> {

    private boolean running = true;

    @Override
    public void run(SourceContext<SensorReading> srcCtx) throws Exception {

        Random rand = new Random();
        int taskIdx = this.getRuntimeContext().getIndexOfThisSubtask();

        String[] sensorIds = new String[10];
        double[] curFTemp = new double[10];
        for (int i = 0; i < 10; i++) {
            sensorIds[i] = "sensor_" + (taskIdx * 10 + i);
            curFTemp[i] = 65 + (rand.nextGaussian() * 20);
        }

        while (running) {
            long curTime = Calendar.getInstance().getTimeInMillis();
            for (int i = 0; i < 10; i++) {
                curFTemp[i] += rand.nextGaussian() * 0.5;
                srcCtx.collect(new SensorReading(sensorIds[i], curTime, curFTemp[i]));
            }

            Thread.sleep(100);
        }
    }

    @Override
    public void cancel() {
        this.running = false;
    }
}
```

使用方法

```java
// 摄入数据流
DataStream[SensorReading] sensorData = env.addSource(new SensorSource)
```

>注意，在我们本教程中，我们一直会使用这个自定义的数据源。

