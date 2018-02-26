# Following these tutorials 
* https://www.tensorflow.org/serving/serving_basic
* https://www.tensorflow.org/serving/serving_advanced


1. Create the `mnist_model` and store under /tmp/
```
python mnist_saved_model.py /tmp/mnist_model
```

2. Start the TFS server

```
tensorflow_model_server --port=9000 --model_name=mnist --model_base_path=/tmp/mnist_model/
```

3. Test the server
```
python mnist_client.py --num_tests=1000 --server=localhost:9000
```

4. Create a better 'mnist_model`
```
python mnist_saved_model.py /tmp/mnist_model --training_iteration=2000 --model_version=2
```

5.  Test the server again and notice the Manager automatically picked up the new model (error % dropped)
```
python mnist_client.py --num_tests=1000 --server=localhost:9000
```