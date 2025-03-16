---


---

<h1 id="regression"><span class="prefix"></span><span class="content">Regression</span><span class="suffix"></span></h1>
<h3 id="python实操"><span class="prefix"></span><span class="content">python实操</span><span class="suffix"></span></h3>
<pre><code>import matplotlib.pyplot as plot
import csv

# Define the objective -----

# Prepare the data -----

csv_file = open('dose_figures.csv')
csv_reader = csv.reader(csv_file)

x_train = [0.0]*20
y_train = [0.0]*20

for i, row in enumerate(csv_reader):
	x_train[i] = float(row[0])
	y_train[i] = float(row[1])

f01 = plot.figure(1)
plot.scatter(x_train, y_train, c='b')
plot.xlabel('mass of drugs injected i.v. (mg)')
plot.ylabel('mass of drugs in brain (\u03BCg)')
ax = plot.gca()
ax.set_xlim(0, 10)
ax.set_ylim(0, 2500)
plot.show()
</code></pre>

