FROM gcr.io/kubeflow-images-public/tensorflow-2.1.0-notebook-cpu:1.0.0
USER root
RUN pip3 install tensorflow-datasets==2.1.0 \
		  boto3==1.18.24 \
		  kubeflow-metadata==0.3.1 \
		  pandas==1.0.3 \
		  protobuf==3.13
ENV NB_PREFIX /
