# NEST e-prop Tutorial Mapping

## 1. Neuron Models Used
- Name (Nest models): 
    - Input "neurons": spike_generator (device) 0-> parrot_neuron (copies spikes, parrot is needed because devices can't be presynaptic for plastic synapses in this set-up), back-end specific plumbing  
    - Recurrent (regular): eprop_iaf_bsshslm_2020
    - Recurrent (adaptive): eprop_iaf_adapt_bsshslm_2020
    - readout: epeprop_readout_bsshslm_2020
    - Target signal generator: step_rate_generator      
- Key parameters:
    - Recurrent neurons include e-prop extras like gamma, surrogate_gradient_function, c_reg, f_target, regular_spike_arrival, etc.
    - Adaptive neuron adds adapt_tau, adaptation, adapt_beta.
    -  Readout neuron has loss="cross_entropy" and uses multiple receptor types (see below).
- State variables/learning-related buffers:
    - Neurons internally maintain e-prop traces / surrogate-gradient related state (not exposed in PyNN normally).
    - Readout exposes things like readout_signal, target_signal via multimeter events (used for evaluation).

## 2. Synapse Models Used
A) Plastic Synapses (Trainable weights)
- Name: `eprop_synapse_bsshslm_2020`
- Used for:
  - Input -> Recurrent
  - Recurrent -> Recurrent
  - Recurrent -> Readout
- Plasticity rule: e-prop (eligibility trace x learning signal)
- Required parameters: 
  - optimiser (Adam config)
  - eta (learning rate)
  - Wmin / Wmax 
  - batch_size
  - average_gradient
  - weight_recorder 

B) Learning Signal Connection
- Name: `eprop_learning_signal_connection_bsshslm_2020`
- Role:
 - Transmits learning signal from readout to recurrent neurons
- Not a weight-learning synapse
- Carries error term

C) Rate connections (Supervision + Softmax)
- Name: `rate_connection_delayed`
- Used for:
  - Target → Readout (teacher signal)
  - Readout ↔ Readout (softmax normalisation)
- Uses receptor_type routing

### D. Static Synapses (Plumbing)

- Name: `static_synapse`
- Used for:
  - spike_generator → parrot_neuron
  - recorders


## 3. Learning signal
- Where is it injected?
    - Injected from:
    Readout neurons -> Recurrent neurons
    - Mechanism:
    Via `eprop_learning_signal_connection_bsshslm_2020`

- How is it scheduled?
    - eprop_update_interval (kernel-level parameter)
    - eprop_learning_window
    - Weight updates happen automatically inside synapse model

Learning signal is separate from forward spike propagation    

## 4. Training Loop Structure
- Simulation phases:
i) Input spike generation
ii) Forward propagation
iii) Readout comparison with target
iv) Learning signal computation
v) Weight update at fixed interval

- Parameter updates:
   - Controlled by:
    - eprop_update_interval
    - eprop_learning_window
    - optimiser parameters
   - No explicit Python training loop
   - Updates occur internally during Nest.Simulate()

- Data collection:
i) multimeters record
  - V_m
  - surrogate_gradient
  - learning_signal
  - readout_signal
  - target_signal
  - error_signal
 ii) Weight recorder tracks synaptic changes 

## 5. Backend-Specific Components
- What cannot be abstracted easily?
    - parrot_neuron required because devices cannot be presynaptic for plastic synapses
    - Forced final spike update to flush optimiser state
    - receptor_type routing for readout softmax 
    - Eligibility traces maintained internally in neuron/synapse model
    - Kernel-level scheduling of weight updates   


## 6. Candidate Abstraction Points
- What should become part of LearningRule API?
    - Rule type (eprop)
    - Learning rate (eta)
    - Optimiser configuration 
    - Weight constraints (Wmin, Wmax) 
    - Batch size
    - Update interval
    - Learning window

- What should remain backend-specific?
    - Eligibility trace implementation 
    - Surrogate gradient implementation
    - Internal optimiser state
    - Device workarounds (parrot)
    - Receptor routing mechanics