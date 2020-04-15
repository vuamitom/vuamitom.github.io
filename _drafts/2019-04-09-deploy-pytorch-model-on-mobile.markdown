




RuntimeError: ONNX export failed: Could not export a broadcasted operation; ONNX likely does not support this form of broadcasting.

Broadcast occurred at:
/home/tamvm/Projects/shapenet/shapenet/layer/shape_layer.py(111): forward
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/nn/modules/module.py(477): _slow_forward
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/nn/modules/module.py(487): __call__
/home/tamvm/Projects/shapenet/shapenet/layer/shape_layer.py(46): forward
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/nn/modules/module.py(477): _slow_forward
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/nn/modules/module.py(487): __call__
/home/tamvm/Projects/shapenet/shapenet/layer/homogeneous_shape_layer.py(66): forward
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/nn/modules/module.py(477): _slow_forward
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/nn/modules/module.py(487): __call__
/home/tamvm/Projects/shapenet/shapenet/networks/network.py(46): forward
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/nn/modules/module.py(477): _slow_forward
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/nn/modules/module.py(487): __call__
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/jit/__init__.py(252): forward
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/nn/modules/module.py(489): __call__
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/jit/__init__.py(197): get_trace_graph
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/onnx/utils.py(192): _trace_and_get_graph_from_model
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/onnx/utils.py(224): _model_to_graph
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/onnx/utils.py(281): _export
/home/tamvm/.virtualenvs/ai/lib/python3.6/site-packages/torch/onnx/__init__.py(22): _export
/home/tamvm/Projects/shapenet/shapenet/scripts/convert_mobile.py(20): convert
/home/tamvm/Projects/shapenet/shapenet/scripts/convert_mobile.py(32): <module>
/usr/lib/python3.6/runpy.py(85): _run_code
/usr/lib/python3.6/runpy.py(193): _run_module_as_main




--> Solution : replace expand with repeat


https://github.com/pytorch/pytorch/issues/12896



Onnx issue 

onnx.onnx_cpp2py_export.checker.ValidationError: No Op or Function registered for ConstantFill with domain_version of 9


https://github.com/facebookresearch/pytext/issues/240

fix by downgrading to onnx 1.3.0 