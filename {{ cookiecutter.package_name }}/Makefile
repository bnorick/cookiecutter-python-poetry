PACKAGE := {{ cookiecutter.package_name }}
PACKAGE_DIR := src/${PACKAGE}
MODULES := $(wildcard $(PACKAGE_DIR)/*.py)

# MAIN TASKS ##################################################################

.PHONY: all
all: doctor format check test ## Run all tasks that determine CI status

.PHONY: dev
dev: install  ## Continuously run all CI tasks when files chanage
	poetry run ptw

.PHONY: ipython
ipython: install
	poetry run ipython --ipython-dir=notebooks --profile-dir=notebooks/.ipython_profile

.PHONY: demo
demo: install
	poetry run nbstripout notebooks/demos/*.ipynb 2>/dev/null || true
	poetry run jupyter notebook --ServerApp.root_dir=notebooks --ServerApp.preferred_dir=notebooks/demos

.PHONY: notebook
notebook: install
	poetry run jupyter notebook --ServerApp.root_dir=notebooks

# SYSTEM DEPENDENCIES #########################################################

.PHONY: bootstrap
bootstrap: ## Attempt to install system dependencies
	asdf plugin add python || asdf plugin update python
	asdf plugin-add poetry https://github.com/asdf-community/asdf-poetry.git || asdf plugin update poetry
	asdf install

.PHONY: doctor
doctor: ## Confirm system dependencies are available
	bin/verchew

# PROJECT DEPENDENCIES ########################################################

VIRTUAL_ENV ?= .venv
DEPENDENCIES := $(VIRTUAL_ENV)/.poetry-$(shell bin/checksum pyproject.toml poetry.lock)

.PHONY: install
install: $(DEPENDENCIES) .cache ## Install project dependencies

$(DEPENDENCIES): poetry.lock
	@ rm -rf $(VIRTUAL_ENV)/.poetry-*
	@ poetry config virtualenvs.in-project true
	poetry install
	@ touch $@

poetry.lock: pyproject.toml
	poetry lock --no-update
	@ touch $@

.cache:
	@ mkdir -p .cache

# CHECKS ######################################################################

.PHONY: format
format: install
	poetry run ruff ${PACKAGE_DIR} tests
	poetry run black $(PACKAGE_DIR) tests
	@ echo

.PHONY: check
check: install format  ## Run formaters, linters, and static analysis
	poetry run mypy ${PACKAGE_DIR} tests
	poetry run deptry --config pyproject.toml ${PACKAGE_DIR} tests

# TESTS #######################################################################

.PHONY: test
test: install  ## Run unit and integration tests
	poetry run pytest --random --doctest-modules

.PHONY: test-repeat
test-repeat: install
	poetry run pytest --count=5 --random --doctest-modules --exitfirst

# RELEASE #####################################################################

DIST_FILES := dist/*.tar.gz dist/*.whl

.PHONY: dist
dist: install $(DIST_FILES)
$(DIST_FILES): $(MODULES) pyproject.toml
	rm -f $(DIST_FILES)
	poetry build

.PHONY: upload
upload: dist  ## Upload the current version to PyPI
	git diff --name-only --exit-code
	poetry publish
	bin/open https://pypi.org/project/$(PACKAGE)

# CLEANUP #####################################################################

.PHONY: clean
clean:  ## Delete all generated and temporary files
	rm -rf .venv

# HELP ########################################################################

.PHONY: help
help: install
	@ grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

.DEFAULT_GOAL := help