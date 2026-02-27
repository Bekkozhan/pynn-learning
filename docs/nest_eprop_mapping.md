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
- Name:
Plastic Synapse

- Plasticity rule:
- Required parameters:

## 3. Learning Signal
- Where is it injected?
- How is it scheduled?

## 4. Training Loop Structure
- Simulation phases:
- Parameter updates:
- Data collection:

## 5. Backend-Specific Components
- What cannot be abstracted easily?

## 6. Candidate Abstraction Points
- What should become part of LearningRule API?
- What should remain backend-specific?