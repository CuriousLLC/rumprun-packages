include ../Makefile.inc

EXPRESS_VERSION=4.13.3

express-$(EXPRESS_VERSION).iso: | express-$(EXPRESS_VERSION)
	genisoimage -l -r -o $@ express-$(EXPRESS_VERSION)

.PHONY: run_express_hello_world
run_express_hello_world: express-$(EXPRESS_VERSION).iso
	rumprun kvm -M 160 -I 'nic,vioif,-net user,hostfwd=tcp::3000-:3000' -W nic,inet,dhcp -i -b express-$(EXPRESS_VERSION).iso,/express ../$(NODE_BIN).bin /express/examples/hello-world/index.js

.PHONY: bundle_express_hello_world
bundle_express_hello_world: | node_modules express-$(EXPRESS_VERSION)
	./node_modules/.bin/webpack --target node --module-bind json ./express-$(EXPRESS_VERSION)/examples/hello-world/index.js hello-world.js
	ln -sf examples/hello-world.js ../rumpmain.js
	make -C ..

.PHONY: run_kvm
run_kvm:
	rumprun kvm -M 160 -I 'nic,vioif,-net user,hostfwd=tcp::3000-:3000' -W nic,inet,dhcp -i ../$(NODE_BIN).bin

ursa.iso: ursa/test.js | node_modules ursa/node_modules/ursa
	genisoimage -l -r -o $@ ursa

NAD=../../../node_modules/.bin/nad
.PHONY: inject_ursa
inject_ursa: | node_modules ursa/node_modules/ursa
	(cd ursa/node_modules/ursa; $(NAD) configure --nodedir ../../../../$(BUILD_DIR); $(NAD) inject)
	NODE_PATH=../examples/ursa/node_modules/ursa/node_modules make -C ..

.PHONY: run_ursa
run_ursa: ursa.iso
	rumprun kvm -M 160 -i -b ursa.iso,/ursa ../$(NODE_BIN).bin /ursa/test.js

node_modules:
	npm install webpack json-loader nad

express-$(EXPRESS_VERSION):
	curl -L https://github.com/strongloop/express/archive/$(EXPRESS_VERSION).tar.gz | tar -zx
	(cd $@ && npm install --production)

ursa/node_modules/ursa:
	(cd ursa && npm install --production)

.PHONY: clean
clean:
	rm -rf *.iso node_modules ursa/node_modules express-* hello-world.js
