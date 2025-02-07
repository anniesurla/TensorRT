.. _use_from_pytorch:

Using Torch-TensorRT Directly From PyTorch
============================================

You will now be able to directly access TensorRT from PyTorch APIs. The process to use this feature
is very similar to the compilation workflow described in :ref:`getting_started_with_python_api`

Start by loading ``torch_tensorrt`` into your application.

.. code-block:: python

    import torch
    import torch_tensorrt


Then given a TorchScript module, you can compile it with TensorRT using the ``torch._C._jit_to_backend("tensorrt", ...)`` API.

.. code-block:: python

    import torchvision.models as models

    model = models.mobilenet_v2(pretrained=True)
    script_model = torch.jit.script(model)

Unlike the ``compile`` API in Torch-TensorRT which assumes you are trying to compile the ``forward`` function of a module
or the ``convert_method_to_trt_engine`` which converts a specified function to a TensorRT engine, the backend API
will take a dictionary which maps names of functions to compile to Compilation Spec objects which wrap the same
sort of dictionary you would provide to ``compile``. For more information on the compile spec dictionary take a look
at the documentation for the Torch-TensorRT ``TensorRTCompileSpec`` API.

.. code-block:: python

    spec = {
        "forward":
            torch_tensorrt.ts.TensorRTCompileSpec({
                "inputs": [torch_tensorrt.Input([1, 3, 300, 300])],
                "enabled_precisions": {torch.float, torch.half},
                "refit": False,
                "debug": False,
                "device": {
                    "device_type": torch_tensorrt.DeviceType.GPU,
                    "gpu_id": 0,
                    "dla_core": 0,
                    "allow_gpu_fallback": True
                },
                "capability": torch_tensorrt.EngineCapability.default,
                "num_avg_timing_iters": 1,
            })
        }

Now to compile with Torch-TensorRT, provide the target module objects and the spec dictionary to ``torch._C._jit_to_backend("tensorrt", ...)``

.. code-block:: python

    trt_model = torch._C._jit_to_backend("tensorrt", script_model, spec)

To run explicitly call the function of the method you want to run (vs. how you can just call on the module itself in standard PyTorch)

.. code-block:: python

    input = torch.randn((1, 3, 300, 300)).to("cuda").to(torch.half)
    print(trt_model.forward(input))
