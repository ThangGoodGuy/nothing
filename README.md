{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Use Apache SystemML and Spark for Machine Learning"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "This notebook introduces Apache SystemML and shows examples of running different forms of Linear Regression (Direct Solve, Batch Gradient Descent and Conjugate Gradient).  The code snippets are adapted from Apache SystemML __<a href=\"http://systemml.apache.org/get-started\" target=\"_blank\" rel=\"noopener noreferrer\">sample notebooks</a>__ and run on Python 2 with Spark 2.1.  Some familiarity with Python and machine learning algorithms is recommended.\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Table of Contents"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "- [Apache SystemML](#systemml)<br>\n",
    "- [Matrix multiplication](#matrix-multiply)<br>\n",
    "- [Linear Regression: direct solve](#direct-solve)<br>\n",
    "- [Linear Regression: \"out-of-the-box\"](#linear-regression)<br>\n",
    "- [Linear Regression: batch gradient descent](#gradient-descent)<br>\n",
    "- [Linear Regression: conjugate gradient](#conjugate-gradient)<br>\n",
    "- [Linear Regression using mllearn interface](#linear-regression-api)<br>\n",
    "- [Learn more](#summary)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<a id=\"systemml\"></a>\n",
    "## Apache SystemML"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "#### Machine Learning library for a scalable distributable solution\n",
    "Apache SystemML provides an optimal workplace for machine learning using big data. It can be run on top of Apache Spark, where it automatically scales your data, line by line, determining whether your code should be run on the driver or an Apache Spark cluster. In the future, SystemML plans to release developments that include additional deep learning with GPU capabilities such as importing and running neural network architectures and pre-trained models for training.\n",
    "\n",
    "Currently, SystemML contains many popular machine learning algorithms (e.g., regression, clustering, classification, descriptive statistics) that can be used out-of-the-box or customized if necessary. Algorithms are written in Python-like or R-like scripting languages intuitive to a Data Scientist familiar with Python or R. Higher-level abstractions that encapsulate well-known algorithms are also provided for a Data Scientist to get started without needing to know the script implementation details.\n",
    "\n",
    "#### Machine learning development challenges\n",
    "Typically a Data Scientist writes and evaluates a machine learning algorithm with small data on a single machine. To apply the algorithm to big data in a clustered environment often requires an experienced Systems Engineer familiar with distributed programming.  However, an algorithm migrated to run on a large scale environment may not produce the same expected results due to unforeseen problems in the original algorithm or errors in converting the algorithm to run on a cluster. In this situation, revising the algorithm ends up being an error-prone, time-consuming process as the Systems Engineer doesn't usually have expertise with the algorithm and the Data Scientist isn't familiar with the distributed programming that Systems Engineers implement.\n",
    "\n",
    "#### Apache SystemML to the rescue\n",
    "Apache SystemML eliminates these algorithm development issues since it creates an optimized program that adapts to data characteristics. With SystemML, a Data Scientist can focus on algorithm development and let Apache SystemML take care of the distributive processing as needed when running with bigger data in a larger environment.\n",
    "\n",
    "#### Ease of use\n",
    "Apache SystemML functionality can be accessed from Python, Scala and Java application programming interfaces.  Batch processing (e.g., spark-submit) is also supported.  Note Apache SystemML is a pre-installed library and available in Data Science Experience with Spark so no separate installation is required.  For example, use pip to show available version."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 1,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Name: systemml\r\n",
      "Version: 0.14.0\r\n",
      "Summary: Apache SystemML is a distributed and declarative machine learning platform.\r\n",
      "Home-page: http://systemml.apache.org/\r\n",
      "Author: Apache SystemML\r\n",
      "Author-email: dev@systemml.incubator.apache.org\r\n",
      "License: Apache 2.0\r\n",
      "Location: /usr/local/src/analytic-libs/spark-2.0/python-2.7\r\n",
      "Requires: numpy, pandas, scipy\r\n"
     ]
    }
   ],
   "source": [
    "!pip show systemml"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<a id=\"matrix-multiply\"></a>\n",
    "## Matrix multiplication"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "In Linear Algebra operations including operands of type Matrix or Vector, its critical from performance perspective to do operation on group of numbers simultaneously. So Matrix (or Vector) operations play a key role in Linear Algebra performance. In the field, there are multiple libraries available for Matrix operations.\n",
    "\n",
    "Apache SystemML contains many __<a href=\"http://apache.github.io/systemml/dml-language-reference.html#built-in-functions\" target=\"_blank\" rel=\"noopener noreferrer\">built-in functions</a>__ for handling linear algebra operations.  Here's a simple example showing how to create a script that generates a random matrix, multiplies the matrix with its transpose, and computes the sum of the output.  \n",
    "\n",
    "#### Matrix multiplication using Numpy of matrix size (1M x 1K)\n",
    "The following code snippet shows numpy being used to do a matrix multiplication. As sufficient memory is available to hold the data, an operation gets performed and produces the result."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "250075486460.0\n"
     ]
    }
   ],
   "source": [
    "import numpy as np\n",
    "\n",
    "a = np.random.rand(10**6, 1000)\n",
    "transMult = np.matmul(a.transpose(),a)\n",
    "sum = np.sum(transMult)\n",
    "print (sum)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "#### Matrix multiplication using Numpy of matrix size (10M x 1K)\n",
    "The following code snippet shows numpy being used to do a matrix multiplication, this time with a matrix size 10M x 1K. This time, the data cannot fit into memory, so the operation gives an error."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "metadata": {},
   "outputs": [
    {
     "ename": "MemoryError",
     "evalue": "",
     "output_type": "error",
     "traceback": [
      "\u001b[0;31m\u001b[0m",
      "\u001b[0;31mMemoryError\u001b[0mTraceback (most recent call last)",
      "\u001b[0;32m<ipython-input-4-9b5663fab7f7>\u001b[0m in \u001b[0;36m<module>\u001b[0;34m()\u001b[0m\n\u001b[1;32m      1\u001b[0m \u001b[0;32mimport\u001b[0m \u001b[0mnumpy\u001b[0m \u001b[0;32mas\u001b[0m \u001b[0mnp\u001b[0m\u001b[0;34m\u001b[0m\u001b[0m\n\u001b[1;32m      2\u001b[0m \u001b[0;34m\u001b[0m\u001b[0m\n\u001b[0;32m----> 3\u001b[0;31m \u001b[0ma\u001b[0m \u001b[0;34m=\u001b[0m \u001b[0mnp\u001b[0m\u001b[0;34m.\u001b[0m\u001b[0mrandom\u001b[0m\u001b[0;34m.\u001b[0m\u001b[0mrand\u001b[0m\u001b[0;34m(\u001b[0m\u001b[0;36m10\u001b[0m\u001b[0;34m**\u001b[0m\u001b[0;36m7\u001b[0m\u001b[0;34m,\u001b[0m \u001b[0;36m1000\u001b[0m\u001b[0;34m)\u001b[0m\u001b[0;34m\u001b[0m\u001b[0m\n\u001b[0m\u001b[1;32m      4\u001b[0m \u001b[0mtransMult\u001b[0m \u001b[0;34m=\u001b[0m \u001b[0mnp\u001b[0m\u001b[0;34m.\u001b[0m\u001b[0mmatmul\u001b[0m\u001b[0;34m(\u001b[0m\u001b[0ma\u001b[0m\u001b[0;34m.\u001b[0m\u001b[0mtranspose\u001b[0m\u001b[0;34m(\u001b[0m\u001b[0;34m)\u001b[0m\u001b[0;34m,\u001b[0m\u001b[0ma\u001b[0m\u001b[0;34m)\u001b[0m\u001b[0;34m\u001b[0m\u001b[0m\n\u001b[1;32m      5\u001b[0m \u001b[0msum\u001b[0m \u001b[0;34m=\u001b[0m \u001b[0mnp\u001b[0m\u001b[0;34m.\u001b[0m\u001b[0msum\u001b[0m\u001b[0;34m(\u001b[0m\u001b[0mtransMult\u001b[0m\u001b[0;34m)\u001b[0m\u001b[0;34m\u001b[0m\u001b[0m\n",
      "\u001b[0;32mmtrand.pyx\u001b[0m in \u001b[0;36mmtrand.RandomState.rand (numpy/random/mtrand/mtrand.c:19746)\u001b[0;34m()\u001b[0m\n",
      "\u001b[0;32mmtrand.pyx\u001b[0m in \u001b[0;36mmtrand.RandomState.random_sample (numpy/random/mtrand/mtrand.c:15541)\u001b[0;34m()\u001b[0m\n",
      "\u001b[0;32mmtrand.pyx\u001b[0m in \u001b[0;36mmtrand.cont0_array (numpy/random/mtrand/mtrand.c:6141)\u001b[0;34m()\u001b[0m\n",
      "\u001b[0;31mMemoryError\u001b[0m: "
     ]
    }
   ],
   "source": [
    "import numpy as np\n",
    "\n",
    "a = np.random.rand(10**7, 1000)\n",
    "transMult = np.matmul(a.transpose(),a)\n",
    "sum = np.sum(transMult)\n",
    "print (sum)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "#### Matrix multiplication using Apache SystemML of matrix size (10M x 1K)\n",
    "The following code snippet shows a matrix multiplication with a matrix size 10M x 1K. This demonstrates that Apache SystemML does not depend just on local memory, but instead leverages the cluster environment to do any operation. This means that a similar operation with the same data size, which cannot be run using numpy, can be done using Apache SystemML. This example also demonstrates the use of Apache SystemML's __<a href=\"http://apache.github.io/systemml/spark-mlcontext-programming-guide\" target=\"_blank\" rel=\"noopener noreferrer\">MLContext Application Programming Interface (API)</a>__ ."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "2.50081577864e+12\n"
     ]
    }
   ],
   "source": [
    "from systemml import MLContext, dml\n",
    "\n",
    "ml = MLContext(sc)\n",
    "\n",
    "script = \"\"\"\n",
    "    X = rand(rows=$nr, cols=1000, sparsity=1.0)\n",
    "    A = t(X) %*% X\n",
    "    s = sum(A)\n",
    "\"\"\"\n",
    "\n",
    "prog = dml(script).input('$nr', 10**7).output('s')\n",
    "s = ml.execute(prog).get('s')\n",
    "print(s)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<a id=\"linear-regression\"></a>\n",
    "## Data generation"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Now let’s generate some synthetic data through a script specified below and run through Apache SystemML. We will use the same data for all use cases described in the notebook below."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "from systemml import MLContext, dml\n",
    "\n",
    "ml = MLContext(sc)\n",
    "\n",
    "script = \"\"\"\n",
    "    numSamples = $numSamples\n",
    "    numFeatures = $numFeatures\n",
    "    sparsity = 0.5\n",
    "    maxFeatureValue = 15\n",
    "    maxWeight = 25\n",
    "    addNoise = 100\n",
    "    fmt = \"csv\"\n",
    "    trainTestRatio = 0.7\n",
    "    trainSize = trainTestRatio * numSamples\n",
    "    testSize = numSamples - trainSize\n",
    "\n",
    "    X = Rand(rows=numSamples, cols=numFeatures, min=-1, max=1, pdf=\"uniform\", seed=0, sparsity=sparsity)\n",
    "    w = Rand(rows=numFeatures, cols=1, min=-1, max=1, pdf=\"uniform\", seed=0)\n",
    "    X = X * maxFeatureValue\n",
    "    w = w * maxWeight\n",
    "    Y = X %*% w\n",
    "\n",
    "    noise = Rand(rows=numSamples, cols=1, pdf=\"normal\", seed=0)\n",
    "    Y = Y + addNoise*noise\n",
    "    \n",
    "    X_train = X[1:trainSize,]\n",
    "    X_test = X[trainSize+1:numSamples,]\n",
    "    y_train = Y[1:trainSize,]\n",
    "    y_test = Y[trainSize+1:numSamples,]\n",
    "\"\"\"\n",
    "\n",
    "prog = dml(script).input('$numSamples', 10**6).input('$numFeatures', 10).output('X_train').output('y_train').output('X_test').output('y_test')\n",
    "X_train_m, y_train_m, X_test_m, y_test_m = ml.execute(prog).get('X_train','y_train','X_test','y_test')\n",
    "X_train = X_train_m.toNumPy()\n",
    "y_train = y_train_m.toNumPy()\n",
    "X_test = X_test_m.toNumPy()\n",
    "y_test = y_test_m.toNumPy()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<a id=\"direct-solve\"></a>\n",
    "## Linear Regression: direct solve\n",
    "\n",
    "This algorithm computes the least squares solution for system of linear equations A %*% w = b i.e., it finds w such that ||A%*%w – b|| is minimized, where A = X'X and b= X'y\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Least squares formulation\n",
    "\\begin{equation*}\n",
    "w* = argminw ||Xw-y||^2 = argminw (y - Xw)'(y - Xw) = argminw (w'(X'X)w - w'(X'y))/2\n",
    "\\end{equation*}\n",
    "\n",
    "where \n",
    "    X is matrix of input features, \n",
    "    y is vector of output value, \n",
    "    w is a vector of learnt weights"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Setting the gradient\n",
    "dw = (X'X)w - (X'y) to 0, w = (X' y)/(X'X) = solve(X'X, X'y)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "The following cell creates and runs a script using Apache SystemML that generates a small Spark job for distributed data processing."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "from systemml import MLContext, dml\n",
    "\n",
    "ml = MLContext(sc)\n",
    "\n",
    "script = \"\"\"\n",
    "    # add constant feature to X to model intercept\n",
    "    X = cbind(X, matrix(1, rows=nrow(X), cols=1))\n",
    "    A = t(X) %*% X\n",
    "    b = t(X) %*% y\n",
    "    w = solve(A, b)\n",
    "    bias = as.scalar(w[nrow(w),1])\n",
    "    w = w[1:nrow(w)-1,]\n",
    "\"\"\"\n",
    "\n",
    "prog = dml(script).input(X=X_train, y=y_train).output('w', 'bias')\n",
    "w, bias = ml.execute(prog).get('w','bias')\n",
    "w = w.toNumPy()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Display the results with matplotlib."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<matplotlib.text.Text at 0x7f14b0eca790>"
      ]
     },
     "execution_count": 8,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f14bfee11d0>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "%matplotlib inline\n",
    "\n",
    "import matplotlib.pyplot as plt\n",
    "import numpy as np\n",
    "\n",
    "chosen_idx = np.random.choice(300000, replace=False, size=10000)\n",
    "plt.scatter(y_test[chosen_idx], np.matmul(X_test[chosen_idx],w)+bias,  color='blue', marker='.', alpha=0.5, s=5)\n",
    "plt.xlabel('Actual Value')\n",
    "plt.ylabel('Predicted Value')"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<a id=\"linear-regression\"></a>\n",
    "## Linear Regression:  \"out-of-the-box\""
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Apache SystemML has a large selection of __<a href=\"https://apache.github.io/systemml/algorithms-reference.html\" target=\"_blank\" rel=\"noopener noreferrer\">algorithm implementations</a>__ that can be used without modifications.  For this example, an existing algorithm will be loaded and executed. "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 9,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "[ 0.00644517]\n"
     ]
    }
   ],
   "source": [
    "from systemml import MLContext, dml, dmlFromResource\n",
    "\n",
    "ml = MLContext(sc)\n",
    "\n",
    "dml_script = dmlFromResource(\"/scripts/algorithms/LinearRegDS.dml\")\n",
    "\n",
    "prog = dml_script.input(X=X_train, y=y_train).input('$icpt',1.0).output('beta_out')\n",
    "w = ml.execute(prog).get('beta_out')\n",
    "w = w.toNumPy()\n",
    "\n",
    "bias = w[w.shape[0]-1]\n",
    "w = w[:w.shape[0]-1]\n",
    "\n",
    "print(bias)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "The results can be displayed graphically using matplotlib as shown below."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<matplotlib.text.Text at 0x7f14b0e41b90>"
      ]
     },
     "execution_count": 10,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\ngg==\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f14bff4cd50>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "%matplotlib inline\n",
    "\n",
    "import matplotlib.pyplot as plt\n",
    "import numpy as np\n",
    "\n",
    "chosen_idx = np.random.choice(300000, replace=False, size=10000)\n",
    "plt.scatter(y_test[chosen_idx], np.matmul(X_test[chosen_idx],w)+bias,  color='blue', marker='.', alpha=0.5, s=5)\n",
    "plt.xlabel('Actual Value')\n",
    "plt.ylabel('Predicted Value')"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<a id=\"gradient-descent\"></a>\n",
    "## Linear Regression: batch gradient descent"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Gradient descent is a first-order iterative optimization algorithm for finding the minimum of a function. To find a local minimum of a function using gradient descent, you can take steps proportional to the negative of the gradient (or of the approximate gradient) of the function at the current point.\n",
    "\n",
    "#### Algorithm\n",
    "`Step 1: Start with an initial point \n",
    "while(not converged) { \n",
    "  Step 2: Compute gradient dw. \n",
    "  Step 3: Compute stepsize alpha.     \n",
    "  Step 4: Update: wnew = wold + alpha*dw \n",
    "}`\n",
    "\n",
    "#### Gradient formula\n",
    "`dw = r = (X'X)w - (X'y)`\n",
    "\n",
    "#### Step size formula\n",
    "`Find number alpha to minimize f(w + alpha*r) \n",
    "alpha = -(r'r)/(r'X'Xr)`\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Create and execute an Apache SystemML script that uses Spark to distribute data processing."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 11,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "from systemml import MLContext, dml\n",
    "\n",
    "ml = MLContext(sc)\n",
    "\n",
    "script = \"\"\"\n",
    "    # add constant feature to X to model intercepts\n",
    "    X = cbind(X, matrix(1, rows=nrow(X), cols=1))\n",
    "    max_iter = 100\n",
    "    w = matrix(0, rows=ncol(X), cols=1)\n",
    "    for(i in 1:max_iter){\n",
    "        XtX = t(X) %*% X\n",
    "        dw = XtX %*%w - t(X) %*% y\n",
    "        alpha = -(t(dw) %*% dw) / (t(dw) %*% XtX %*% dw)\n",
    "        w = w + dw*alpha\n",
    "    }\n",
    "    bias = as.scalar(w[nrow(w),1])\n",
    "    w = w[1:nrow(w)-1,]    \n",
    "\"\"\"\n",
    "\n",
    "prog = dml(script).input(X=X_train, y=y_train).output('w').output('bias')\n",
    "w, bias = ml.execute(prog).get('w', 'bias')\n",
    "w = w.toNumPy()\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Visualize the results with matplotlib."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 12,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<matplotlib.text.Text at 0x7f14abf70ed0>"
      ]
     },
     "execution_count": 12,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f14b0eacd50>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "%matplotlib inline\n",
    "\n",
    "import matplotlib.pyplot as plt\n",
    "import numpy as np\n",
    "\n",
    "chosen_idx = np.random.choice(300000, replace=False, size=10000)\n",
    "plt.scatter(y_test[chosen_idx], np.matmul(X_test[chosen_idx],w)+bias,  color='blue', marker='.', alpha=0.5, s=5)\n",
    "plt.xlabel('Actual Value')\n",
    "plt.ylabel('Predicted Value')"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<a id=\"conjugate-gradient\"></a>\n",
    "## Linear Regression: conjugate gradient"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "A problem with gradient descent is that it takes very similar directions many times.  Modifying the original gradient descent algorithm with an additional step to enforcing conjugacy resolves this issue.\n",
    "\n",
    "`Step 1: Start with an initial point \n",
    "while(not converged) {\n",
    "   Step 2: Compute gradient dw.\n",
    "   Step 3: Compute stepsize alpha.\n",
    "   Step 4: Compute next direction p by enforcing conjugacy with previous direction.\n",
    "   Step 5: Update: w_new = w_old + alpha*p\n",
    "}`\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Define the algorithm in Apache SystemML's scripting language and execute the corresponding program using __<a href=\"http://apache.github.io/systemml/spark-mlcontext-programming-guide\" target=\"_blank\" rel=\"noopener noreferrer\">MLContext Application Programming Interface (API)</a>__."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 13,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "from systemml import MLContext, dml\n",
    "\n",
    "ml = MLContext(sc)\n",
    "\n",
    "script = \"\"\"\n",
    "    # add constant feature to X to model intercepts\n",
    "    X = cbind(X, matrix(1, rows=nrow(X), cols=1))\n",
    "    m = ncol(X); i = 1; \n",
    "    max_iter = 20;\n",
    "    w = matrix (0, rows = m, cols = 1); # initialize weights to 0\n",
    "    dw = - t(X) %*% y; p = - dw;        # dw = (X'X)w - (X'y)\n",
    "    norm_r2 = sum (dw ^ 2); \n",
    "    for(i in 1:max_iter) {\n",
    "        q = t(X) %*% (X %*% p)\n",
    "        alpha = norm_r2 / sum (p * q);  # Minimizes f(w - alpha*r)\n",
    "        w = w + alpha * p;              # update weights\n",
    "        dw = dw + alpha * q;           \n",
    "        old_norm_r2 = norm_r2; norm_r2 = sum (dw ^ 2);\n",
    "        p = -dw + (norm_r2 / old_norm_r2) * p; # next direction - conjugacy to previous direction\n",
    "        i = i + 1;\n",
    "    }\n",
    "    bias = as.scalar(w[nrow(w),1])\n",
    "    w = w[1:nrow(w)-1,]    \n",
    "\"\"\"\n",
    "\n",
    "prog = dml(script).input(X=X_train, y=y_train).output('w').output('bias')\n",
    "w, bias = ml.execute(prog).get('w','bias')\n",
    "w = w.toNumPy()\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 14,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<matplotlib.text.Text at 0x7f14abe47050>"
      ]
     },
     "execution_count": 14,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\nTkSuQmCC\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f14abf0a710>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "%matplotlib inline\n",
    "\n",
    "import matplotlib.pyplot as plt\n",
    "import numpy as np\n",
    "\n",
    "chosen_idx = np.random.choice(300000, replace=False, size=10000)\n",
    "plt.scatter(y_test[chosen_idx], np.matmul(X_test[chosen_idx],w)+bias,  color='blue', marker='.', alpha=0.5, s=5)\n",
    "plt.xlabel('Actual Value')\n",
    "plt.ylabel('Predicted Value')"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<a id=\"linear-regression-api\"></a>\n",
    "## Linear Regression using mllearn interface"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Python developers can use the __<a href=\"https://apache.github.io/systemml/python-reference#mllearn-api\" target=\"_blank\" rel=\"noopener noreferrer\">mllearn interface</a>__ from Apache SystemML to invoke algorithms with scikit-learn. You will notice that this mllearn interface from Apache SystemML is similar to scikit-learn interface."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "The next cell runs __<a href=\"https://apache.github.io/systemml/algorithms-regression.html#linear-regression\" target=\"_blank\" rel=\"noopener noreferrer\">Linear Regression</a>__ against the sample data taking approximately sixteen Spark jobs to complete processing.  Since the number of features in the sample dataset is small, Linear Regression Direct Solve is used with `solver='direct-solve'`.  To switch to Linear Regression Conjugate Gradient, specify `solver='newton-cg'`."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Apache SystemML code to do linear regression using mllearn interface"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 15,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "from systemml.mllearn import LinearRegression\n",
    "\n",
    "# Create Linear Regression object through sklearn\n",
    "regr = LinearRegression(spark, solver='direct-solve', transferUsingDF=True)\n",
    "\n",
    "# Train the model using the training sets through sklearn\n",
    "regr.fit(X_train, y_train)\n",
    "\n",
    "# Make predictions using the testing set through sklearn\n",
    "mllearn_predicted = regr.predict(X_test)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Utilities from sklearn can be used to (R^2) score the results as shown below."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 16,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Linear Regression score: 1.000000\n"
     ]
    }
   ],
   "source": [
    "from sklearn import linear_model\n",
    "from sklearn.metrics import r2_score\n",
    "\n",
    "# Create Linear Regression object through scikit-learn\n",
    "sklearn_regr = linear_model.LinearRegression()\n",
    "\n",
    "# Train the model using the training sets through scikit-learn\n",
    "sklearn_regr.fit(X_train, y_train)\n",
    "\n",
    "# Make predictions using the testing set through scikit-learn\n",
    "sklearn_predicted = sklearn_regr.predict(X_test)\n",
    "\n",
    "# Calculate R^2 score  using sklearn by passing predictions obtained through mllearn and scikit-learn\n",
    "score = r2_score(sklearn_predicted, mllearn_predicted)\n",
    "print('Linear Regression score: %f' % score)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<a id=\"summary\"></a>\n",
    "## Learn more"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {
    "collapsed": true
   },
   "source": [
    "Visit __<a href=\"http://systemml.apache.org/\" target=\"_blank\" rel=\"noopener noreferrer\">Apache SystemML</a>__ for additional examples and documentation.  For questions or requests for more information, please use the __<a href=\"http://apache.github.io/systemml/contributing-to-systemml#development-mailing-list\" target=\"_blank\" rel=\"noopener noreferrer\">development mailing list</a>__.  To get involved, see __<a href=\"http://apache.github.io/systemml/contributing-to-systemml\" target=\"_blank\" rel=\"noopener noreferrer\">contributing to Apache SystemML</a>__ for details."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<a id=\"authors\"></a>\n",
    "### Authors"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "This notebook is built upon __<a href=\"https://github.com/apache/systemml/graphs/contributors\" target=\"_blank\" rel=\"noopener noreferrer\">contributions from developers</a>__ of Apache SystemML.  **Arvind Surve** is an architect at IBM in San Jose, California.  He has been an active committer for Apache SystemML since it was initially contributed to open source and has written a __<a href=\"http://www.spark.tc/tutorial-declarative-ml/\" target=\"_blank\" rel=\"noopener noreferrer\">Declarative Machine Learning Tutorial</a>__ with Apache SystemML.  **Glenn Weidner** is a software developer at IBM in San Francisco, California.  He's a member of the __<a href=\"http://www.spark.tc/\" target=\"_blank\" rel=\"noopener noreferrer\">Spark Technology Center</a>__ and also a committer for Apache SystemML."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {
    "collapsed": true
   },
   "source": [
    "<hr>\n",
    "Copyright &copy; IBM Corp. 2017. This notebook and its source code are released under the terms of the MIT License."
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 2 with Spark 2.1",
   "language": "python",
   "name": "python2-spark21"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 2
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython2",
   "version": "2.7.11"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 1
}

