# Tutorial-TensorFlowServing
Playing with TensorFlow Serving

# About TensorFlow Serving (TFS)
* https://www.tensorflow.org/serving
* https://www.tensorflow.org/serving/architecture_overview

TFS is a flexible, high-performance serving system for machine learning models, designed for production environments. TFS makes it easy to deploy new algorithms and experiments, while keeping the same server architecture and APIs. TFS provides out-of-the-box integration with TensorFlow models, but can be easily extended to serve other types of models and data.

## Key Concepts

### Servables
Servables are the central abstraction in TFS. They are the underlying objects that clients use to perform computation. The size of a Servable is flexible - can be of any type and interface. Servables do not manage their own lifecycle (managed by Managers/Loaders)

Typical servables include...
- a TF SavedModelBundle (`tensorflow::Session`)
- a lookup table for embedding or vocab lookups

 ### Servable Versions
 TFS can handle one or more versions of a servable over the lifetime of a single server instance --> can load  new algo configs and data over time. More than 1 versions could be loaded concurrently to support gradual rollout and exp. 

 ### Servable Streams
 The sequence of versions of a servable, sorted by increasing version #.

 ### Models
 TFS represents a model as one or more servables. A ML model may include one or more algos (including learned weights) and lookup or embedding tables.

Can represent a composite model as either
1. multiple independent servables
2. single composite servable 

A servable may also correspond to a fraction of a model. (e.g. a large lookup table sharded across multiple TFS)

 ### Loaders
Manage servable's life cycle. Loaders standardize the APIs for loading and unloading a servable (independent from specific learning algos, data, or product use-case).

 ### Sources
Plugin modules that find and provide servables. Each source provides zero or more servable streams. 

### Aspired Versions
Represent the set of servable versions that should be loaded and ready. Sources communicate this set of servable versions for a single servable stream at a time. 

 ### Managers
Handle the full lifecycle of Servables - loading, serving, unloading. Managers listen to Sources and track all versions. Provide a simple, narrow interface - `GetServableHandle()` - for clients to access loaded servable instances.

 ### Core
 Manages (via standard TFS APIs) the servable lifecycles and metrics. Treats servables and loaders as opaque objects. 

## Example 
For example, say a Source represents a TensorFlow graph with frequently updated model weights. The weights are stored in a file on disk.

1. The Source detects a new version of the model weights. It creates a Loader that contains a pointer to the model data on disk.
2. The Source notifies the Dynamic Manager of the Aspired Version.
3. The Dynamic Manager applies the Version Policy and decides to load the new version.
4. The Dynamic Manager tells the Loader that there is enough memory. The Loader instantiates the TensorFlow graph with the new weights.
5. A client requests a handle to the latest version of the model, and the Dynamic Manager returns a handle to the new version of the Servable.