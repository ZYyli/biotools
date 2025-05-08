---


---

<p>如果后续训练想<strong>限制使用特定 GPU</strong>，比如只用 0 号 GPU，可以在代码开头加：<br>
<code>python import tensorflow as tf tf.config.set_visible_devices(tf.config.list_physical_devices('GPU')[0], 'GPU')</code></p>
<p>或者用环境变量：<br>
<code>bash CUDA_VISIBLE_DEVICES=0 python your_script.py</code></p>

