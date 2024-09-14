build: Dockerfile
	docker build . -t pycse-qe


push:
	docker tag pycse-qe jkitchin/pycse-qe:latest
	docker push jkitchin/pycse-qe:latest

run:
	./pycse-qe.sh

