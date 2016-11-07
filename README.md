[![Build Status](https://travis-ci.org/hammerlab/survivalstan.svg?branch=setup-travis)](https://travis-ci.org/hammerlab/survivalstan) 
[![Coverage Status](https://coveralls.io/repos/github/hammerlab/survivalstan/badge.svg?branch=master)](https://coveralls.io/github/hammerlab/survivalstan?branch=master)
[![PyPI version](https://badge.fury.io/py/survivalstan.svg)](https://badge.fury.io/py/survivalstan)

survivalstan: Survival Models in Stan
===============================

author: Jacki Novik

Overview
--------

Library of Stan Models for Survival Analysis

Installation / Usage
--------------------

Once we push this repo to pypi, you will be able to install using pip, as:

    $ pip install survivalstan ## (not yet set up)


Or, you can clone the repo:

    $ git clone https://github.com/hammerlab/survivalstan.git
    $ pip install .

Contributing
------------

Details to come. For now, please do not hesitate to contribute if you would like. 

Example
-------

There are several examples included in the [example-notebooks](), roughly one corresponding to each model.

If you are not sure where to start, [Test pem_survival_model with simulated data.ipynb](https://github.com/hammerlab/survivalstan/blob/master/example-notebooks/Test%20pem_survival_model%20with%20simulated%20data.ipynb) contains the most explanatory text. Many of the other notebooks are sparse on explanation, but do illustrate variations on the different models.

For basic usage:

```
import survivalstan
import stanity
import seaborn as sb
import matplotlib.pyplot as plt
import statsmodels

## load flchain test data from R's `survival` package
dataset = statsmodels.datasets.get_rdataset(package = 'survival', dataname = 'flchain' )
d  = dataset.data.query('futime > 7')
d.reset_index(level = 0, inplace = True)

## e.g. fit Weibull survival model
testfit_wei = survivalstan.fit_stan_survival_model(
	model_cohort = 'Weibull model',
	model_code = survivalstan.models.weibull_survival_model,
	df = d,
	time_col = 'futime',
	event_col = 'death',
	formula = 'age + sex',
	iter = 3000,
	chains = 4,
	make_inits = survivalstan.make_weibull_survival_model_inits
	)

## coefplot for Weibull coefficient estimates
sb.boxplot(x = 'value', y = 'variable', data = testfit_wei['coefs'])

## or, use plot_coefs
survivalstan.utils.plot_coefs([testfit_wei])

## print summary of MCMC draws from posterior for each parameter
print(testfit_wei['fit'])


## e.g. fit Piecewise-exponential survival model 
dlong = survivalstan.prep_data_long_surv(d, time_col = 'futime', event_col = 'death')
testfit_pem = survivalstan.fit_stan_survival_model(
	model_cohort = 'PEM model',
	model_code = survivalstan.models.pem_survival_model,
	df = dlong,
	sample_col = 'index',
	timepoint_end_col = 'end_time',
	event_col = 'end_failure',
	formula = 'age + sex',
	iter = 3000,
	chains = 4,
	)

## print summary of MCMC draws from posterior for each parameter
print(testfit_pem['fit'])

## coefplot for PEM model results
sb.boxplot(x = 'value', y = 'variable', data = testfit_pem['coefs'])

## e.g. compare models using PSIS-LOO
stanity.loo_compare(testfit_wei['loo'], testfit_pem['loo'])

## compare coefplots 
sb.boxplot(x = 'value', y = 'variable', hue = 'model_cohort',
    data = testfit_pem['coefs'].append(testfit_wei['coefs']))
plt.legend(bbox_to_anchor=(1.05, 1), loc=2, borderaxespad=0.)

## (or, use survivalstan.utils.plot_coefs)
survivalstan.utils.plot_coefs([testfit_wei, testfit_pem])

```


