:orphan:

.. _checkpointing_expert:

################################
Distributed checkpoints (expert)
################################

Generally, the bigger your model is, the longer it takes to save a checkpoint to disk.
With distributed checkpoints (sometimes called sharded checkpoints), you can save and load the state of your training script with multiple GPUs or nodes more efficiently, avoiding memory issues.


----


*****************************
Save a distributed checkpoint
*****************************

The distributed checkpoint format can be enabled when you train with the :doc:`FSDP strategy <../advanced/model_parallel/fsdp>`.

.. code-block:: python

    import lightning as L
    from lightning.pytorch.strategies import FSDPStrategy

    # 1. Select the FSDP strategy and set the sharded/distributed checkpoint format
    strategy = FSDPStrategy(state_dict_type="sharded")

    # 2. Pass the strategy to the Trainer
    trainer = L.Trainer(devices=2, strategy=strategy, ...)

    # 3. Run the trainer
    trainer.fit(model)


With ``state_dict_type="sharded"``, each process/GPU will save its own file into a folder at the given path.
This reduces memory peaks and speeds up the saving to disk.

.. collapse:: Full example

    .. code-block:: python

        import lightning as L
        from lightning.pytorch.strategies import FSDPStrategy
        from lightning.pytorch.demos import LightningTransformer

        model = LightningTransformer()

        strategy = FSDPStrategy(state_dict_type="sharded")
        trainer = L.Trainer(
            accelerator="cuda",
            devices=4,
            strategy=strategy,
            max_steps=3,
        )
        trainer.fit(model)


    Check the contents of the checkpoint folder:

    .. code-block:: bash

        ls -a lightning_logs/version_0/checkpoints/epoch=0-step=3.ckpt/

    .. code-block::

        epoch=0-step=3.ckpt/
        ├── __0_0.distcp
        ├── __1_0.distcp
        ├── __2_0.distcp
        ├── __3_0.distcp
        ├── .metadata
        └── meta.pt

    The ``.distcp`` files contain the tensor shards from each process/GPU. You can see that the size of these files
    is roughly 1/4 of the total size of the checkpoint since the script distributes the model across 4 GPUs.


----


*****************************
Load a distributed checkpoint
*****************************

You can easily load a distributed checkpoint in Trainer if your script uses :doc:`FSDP <../advanced/model_parallel/fsdp>`.

.. code-block:: python

    import lightning as L
    from lightning.pytorch.strategies import FSDPStrategy

    # 1. Select the FSDP strategy and set the sharded/distributed checkpoint format
    strategy = FSDPStrategy(state_dict_type="sharded")

    # 2. Pass the strategy to the Trainer
    trainer = L.Trainer(devices=2, strategy=strategy, ...)

    # 3. Set the checkpoint path to load
    trainer.fit(model, ckpt_path="path/to/checkpoint")

Note that you can load the distributed checkpoint even if the world size has changed, i.e., you are running on a different number of GPUs than when you saved the checkpoint.

.. collapse:: Full example

    .. code-block:: python

        import lightning as L
        from lightning.pytorch.strategies import FSDPStrategy
        from lightning.pytorch.demos import LightningTransformer

        model = LightningTransformer()

        strategy = FSDPStrategy(state_dict_type="sharded")
        trainer = L.Trainer(
            accelerator="cuda",
            devices=2,
            strategy=strategy,
            max_steps=5,
        )
        trainer.fit(model, ckpt_path="lightning_logs/version_0/checkpoints/epoch=0-step=3.ckpt")


.. important::

    If you want to load a distributed checkpoint into a script that doesn't use FSDP (or Trainer at all), then you will have to :ref:`convert it to a single-file checkpoint first <Convert dist-checkpoint>`.


----


.. _Convert dist-checkpoint:

********************************
Convert a distributed checkpoint
********************************

Coming soon.
