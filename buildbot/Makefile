ifndef RIAK_DIR
$(error RIAK_DIR is not set)
endif

unexport PYENV_VERSION
unexport LANG
unexport LC_ADDRESS
unexport LC_COLLATE
unexport LC_CTYPE
unexport LC_IDENTIFICATION
unexport LC_MEASUREMENT
unexport LC_MESSAGES
unexport LC_MONETARY
unexport LC_NAME
unexport LC_NUMERIC
unexport LC_PAPER
unexport LC_TELEPHONE
unexport LC_TIME

PROJDIR = $(realpath $(CURDIR)/..)
TOOLS_DIR = $(PROJDIR)/tools/devrel
CA_DIR = $(PROJDIR)/tools/test-ca
RIAK_CONF = $(RIAK_DIR)/etc/riak.conf
ADV_CONF = $(RIAK_DIR)/etc/advanced.config
RIAK_ADMIN = $(RIAK_DIR)/bin/riak-admin

preconfigure:
	$(TOOLS_DIR)/gen-riak-conf $(RIAK_CONF) 8098 8087 18098 $(CA_DIR)/certs/cacert.pem $(CA_DIR)/certs/riak-test-cert.pem $(CA_DIR)/private/riak-test-key.pem
	$(TOOLS_DIR)/gen-adv-conf $(ADV_CONF)

configure:
	$(TOOLS_DIR)/riak-cluster-config $(RIAK_ADMIN) 8098 true true

compile:
	@echo NO-OP

lint:
	@pip install --upgrade pep8 flake8
	@cd ..; flake8 --exclude=riak/pb riak *.py

test: setup test_normal test_security

test_normal:
	@echo "Testing Riak Python Client (without security)"
	@$(RIAK_ADMIN) security disable
	@RIAK_TEST_PROTOCOL='pbc' RIAK_TEST_PB_PORT=8087 RUN_BTYPES=1 RUN_CLIENT=1 RUN_MAPREDUCE=1 RUN_KV=1 RUN_RESOLVE=1 RUN_YZ=1 RUN_DATATYPES=1 RUN_INDEXES=1 ./tox_runner.sh ..
	@RIAK_TEST_PROTOCOL='http' RIAK_TEST_HTTP_PORT=8098 RUN_BTYPES=1 RUN_CLIENT=1 RUN_MAPREDUCE=1 RUN_KV=1 RUN_RESOLVE=1 RUN_YZ=1 RUN_DATATYPES=1 RUN_INDEXES=1 ./tox_runner.sh ..

test_security:
	@echo "Testing Riak Python Client (with security)"
	@$(RIAK_ADMIN) security enable
	@RIAK_TEST_PROTOCOL='pbc' RIAK_TEST_PB_PORT=8087 RUN_BTYPES=1 RUN_CLIENT=1 RUN_KV=1 RUN_YZ=1 RUN_INDEXES=1 RUN_SECURITY=1 ./tox_runner.sh ..
	@RIAK_TEST_PROTOCOL='http' RIAK_TEST_HTTP_PORT=18098 RUN_BTYPES=1 RUN_CLIENT=1 RUN_KV=1 RUN_YZ=1 RUN_INDEXES=1 RUN_SECURITY=1 ./tox_runner.sh ..

test_timeseries:
	@echo "Testing Riak Python Client (timeseries)"
	@$(RIAK_ADMIN) security disable
	@RIAK_TEST_PROTOCOL='pbc' RIAK_TEST_PB_PORT=8087 RUN_CLIENT=1 RUN_KV=1 RUN_INDEXES=1 RUN_TIMESERIES=1 ./tox_runner.sh ..

test_hll:
	@echo "Testing Riak Python Client (HLL datatype)"
	@$(RIAK_ADMIN) security disable
	@RIAK_TEST_PROTOCOL='pbc' RIAK_TEST_PB_PORT=8087 RUN_DATATYPES=1 RUN_DATATYPE_HLL=1 ./tox_runner.sh ..
	@RIAK_TEST_PROTOCOL='http' RIAK_TEST_HTTP_PORT=8098 RUN_DATATYPES=1 RUN_DATATYPE_HLL=1 ./tox_runner.sh ..

setup:
	./tox_setup.sh
