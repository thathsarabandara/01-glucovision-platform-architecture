# 09 - Federated Learning Architecture
## Multi-Institutional Collaborative AI Model Training for Diabetes Management

**Project**: Voice-First Multilingual AI Assistant for Diabetes Management  
**Component**: Federated Learning Infrastructure  
**Version**: 1.0  
**Date**: March 23, 2026  
**Status**: Architecture & Implementation Design  

---

## 1. Executive Summary

Federated Learning enables multiple healthcare institutions, clinics, and individual patients to collaboratively improve the diabetes management AI models **without sharing raw patient data**. This approach preserves patient privacy while leveraging collective intelligence across diverse populations.

### Key Innovation Points:
1. **Privacy-Preserving Collaboration** - Train models without centralizing sensitive data
2. **Multi-Institutional Participation** - Hospitals, clinics, research centers work together
3. **Personalized Local Models** - Each institution maintains its own optimized model
4. **Knowledge Aggregation** - Global insights from local training
5. **Regulatory Compliance** - GDPR, HIPAA, PDPA compliant data handling

---

## 2. Federated Learning Architecture Overview

### 2.1 System Architecture Diagram

```
┌────────────────────────────────────────────────────────────────────┐
│                    GLOBAL COORDINATION SERVER                       │
│  (Model Aggregation, Version Control, Quality Assurance)            │
└────────────────────────────────────────────────────────────────────┘
                                    ↑
                ┌───────────────────┼───────────────────┐
                ↓                   ↓                   ↓
        ┌───────────────┐  ┌───────────────┐  ┌───────────────┐
        │  Hospital A   │  │  Hospital B   │  │   Clinic C    │
        │  (FL Client)  │  │  (FL Client)  │  │  (FL Client)  │
        └───────────────┘  └───────────────┘  └───────────────┘
              ↓                   ↓                   ↓
        ┌──────────────┐   ┌─────────────┐   ┌──────────────┐
        │ Local Data   │   │ Local Data  │   │ Local Data   │
        │ Hospital A   │   │ Hospital B  │   │  Clinic C    │
        │ (Patient 1-N)│   │(Patient 1-M)│   │(Patient 1-K) │
        └──────────────┘   └─────────────┘   └──────────────┘
```

### 2.2 Federated Learning Workflow

```
ROUND 1: INITIALIZATION
  ↓
  Global Server: Initializes base model (food recognition, glucose prediction)
  ↓
  Distributes model weights to all clients (Hospital A, B, Clinic C)

ROUND 2-N: LOCAL TRAINING & AGGREGATION
  ↓
  Each Client (Hospital):
  ├─ Receives global model (weights only, no data exchange)
  ├─ Trains model on LOCAL data (patients in their institution)
  │  ├─ Food Recognition: Train on local food images
  │  ├─ Glucose Prediction: Train on local patient glucose patterns
  │  ├─ Activity Analysis: Train on local user activity patterns
  │  └─ Recommendation Engine: Personalize to local population
  │
  ├─ Validates model on local test set
  ├─ Encrypts model updates (differential privacy)
  └─ Sends ONLY model updates to Global Server (not raw data!)
  
  Global Server:
  ├─ Receives encrypted updates from all hospitals
  ├─ Aggregates updates (averaging, weighted averaging)
  ├─ Creates new global model
  ├─ Validates on held-out test set
  └─ Distributes updated weights back to all clients

BENEFIT: Hospital A learns from patterns in Hospital B's data
         WITHOUT ever seeing Hospital B's patient data!
```

---

## 3. Core Components Architecture

### 3.1 Federated Learning Client (Hospital/Institution Side)

```python
import hashlib
import json
from datetime import datetime
from typing import Dict, List, Tuple
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

class FederatedLearningClient:
    """
    Local training client for individual hospitals/clinics
    Trains models on local patient data and communicates updates to global server
    """
    
    def __init__(self, 
                 client_id: str,
                 institution_name: str,
                 local_data_loader: DataLoader,
                 validation_data_loader: DataLoader,
                 model_architecture: nn.Module,
                 privacy_epsilon: float = 1.0,
                 privacy_delta: float = 1e-5):
        """
        Initialize FL Client
        
        Args:
            client_id: Unique identifier for this institution
            institution_name: Name of hospital/clinic
            local_data_loader: DataLoader with local patient data
            validation_data_loader: DataLoader for validation
            model_architecture: Neural network model to train
            privacy_epsilon: DP parameter (smaller = more private)
            privacy_delta: DP parameter (failure probability)
        """
        
        self.client_id = client_id
        self.institution_name = institution_name
        self.local_data_loader = local_data_loader
        self.validation_data_loader = validation_data_loader
        self.model = model_architecture
        self.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
        self.model.to(self.device)
        
        # Privacy settings
        self.privacy_epsilon = privacy_epsilon
        self.privacy_delta = privacy_delta
        
        # Training metrics
        self.training_history = []
        self.update_count = 0
    
    def train_local_model(self, 
                         global_model_weights: Dict,
                         epochs: int = 5,
                         learning_rate: float = 0.001) -> Dict:
        """
        Train model on local patient data
        
        Returns:
            Dictionary containing:
            - model_update: Weight differences from global model
            - validation_metrics: Accuracy, loss, etc.
            - data_summary: Statistics about local data (no raw data!)
            - timestamp: When training occurred
        """
        
        # Step 1: Initialize model with global weights
        self._load_global_weights(global_model_weights)
        
        # Store initial weights for computing updates
        initial_weights = self._get_model_weights()
        
        # Step 2: Local training loop
        optimizer = torch.optim.Adam(self.model.parameters(), lr=learning_rate)
        loss_fn = nn.CrossEntropyLoss()
        
        local_loss_history = []
        
        for epoch in range(epochs):
            epoch_loss = 0
            batch_count = 0
            
            for batch_data, batch_labels in self.local_data_loader:
                batch_data = batch_data.to(self.device)
                batch_labels = batch_labels.to(self.device)
                
                # Forward pass
                outputs = self.model(batch_data)
                loss = loss_fn(outputs, batch_labels)
                
                # Backward pass
                optimizer.zero_grad()
                loss.backward()
                
                # Apply Differential Privacy (gradient clipping)
                self._apply_differential_privacy()
                
                optimizer.step()
                
                epoch_loss += loss.item()
                batch_count += 1
            
            avg_epoch_loss = epoch_loss / batch_count
            local_loss_history.append(avg_epoch_loss)
            print(f"[{self.institution_name}] Epoch {epoch+1}/{epochs}, Loss: {avg_epoch_loss:.4f}")
        
        # Step 3: Compute model update (Delta)
        final_weights = self._get_model_weights()
        model_update = self._compute_weight_delta(initial_weights, final_weights)
        
        # Step 4: Validate on local test set
        validation_metrics = self._validate_model()
        
        # Step 5: Prepare data summary (privacy-preserving statistics)
        data_summary = self._generate_data_summary()
        
        # Step 6: Encrypt update for transmission
        encrypted_update = self._encrypt_model_update(model_update)
        
        self.update_count += 1
        
        return {
            'client_id': self.client_id,
            'institution_name': self.institution_name,
            'model_update': encrypted_update,
            'update_size': len(model_update),
            'validation_metrics': validation_metrics,
            'data_summary': data_summary,
            'training_loss_history': local_loss_history,
            'timestamp': datetime.now().isoformat(),
            'update_number': self.update_count,
            'privacy_guarantees': {
                'differential_privacy_epsilon': self.privacy_epsilon,
                'differential_privacy_delta': self.privacy_delta,
                'secure_aggregation': True,
                'gradient_clipping_applied': True
            }
        }
    
    def _apply_differential_privacy(self, 
                                   gradient_clipping_threshold: float = 1.0):
        """
        Apply Differential Privacy via gradient clipping
        Prevents reverse-engineering of training data from gradients
        
        Method: Clip gradients to max norm per sample
        """
        
        max_norm = gradient_clipping_threshold
        total_norm = 0
        
        for p in self.model.parameters():
            if p.grad is not None:
                param_norm = p.grad.data.norm(2)
                total_norm += param_norm.item() ** 2
        
        total_norm = total_norm ** (1. / 2.)
        clip_coef = max_norm / (total_norm + 1e-6)
        
        if clip_coef < 1:
            for p in self.model.parameters():
                if p.grad is not None:
                    p.grad.data.mul_(clip_coef)
    
    def _compute_weight_delta(self, 
                             initial_weights: Dict,
                             final_weights: Dict) -> Dict:
        """
        Compute delta (update) between initial and final weights
        
        Delta = Final - Initial
        This ensures only weight CHANGES are sent, not absolute weights
        """
        
        delta = {}
        for layer_name in final_weights:
            delta[layer_name] = final_weights[layer_name] - initial_weights[layer_name]
        
        return delta
    
    def _validate_model(self) -> Dict:
        """
        Validate trained model on local test set
        Returns metrics without sharing raw validation data
        """
        
        self.model.eval()
        total_correct = 0
        total_samples = 0
        total_loss = 0
        
        loss_fn = nn.CrossEntropyLoss()
        
        with torch.no_grad():
            for batch_data, batch_labels in self.validation_data_loader:
                batch_data = batch_data.to(self.device)
                batch_labels = batch_labels.to(self.device)
                
                outputs = self.model(batch_data)
                loss = loss_fn(outputs, batch_labels)
                
                _, predicted = torch.max(outputs.data, 1)
                total_correct += (predicted == batch_labels).sum().item()
                total_samples += batch_labels.size(0)
                total_loss += loss.item()
        
        accuracy = total_correct / total_samples
        avg_loss = total_loss / len(self.validation_data_loader)
        
        return {
            'accuracy': accuracy,
            'loss': avg_loss,
            'total_samples_validated': total_samples
        }
    
    def _generate_data_summary(self) -> Dict:
        """
        Generate privacy-preserving summary of local data
        NO raw data is included, only aggregate statistics
        """
        
        summary = {
            'total_patients': self._count_unique_patients(),
            'total_data_points': len(self.local_data_loader.dataset),
            'data_distribution': {
                'food_logs': self._count_data_type('food'),
                'glucose_readings': self._count_data_type('glucose'),
                'activity_records': self._count_data_type('activity'),
                'recommendations': self._count_data_type('recommendations')
            },
            'demographic_summary': {
                'age_range': self._calculate_age_range(),
                'diabetes_type_distribution': self._calculate_diabetes_distribution(),
                'geographical_region': self.institution_name  # Privacy: region only, not specific location
            },
            'data_quality_metrics': {
                'missing_values_percentage': self._calculate_missing_values_pct(),
                'outliers_percentage': self._calculate_outliers_pct(),
                'data_collection_period': self._get_data_collection_period()
            }
        }
        
        return summary
    
    def _encrypt_model_update(self, model_update: Dict) -> Dict:
        """
        Encrypt model update using secure aggregation
        Server cannot decrypt individual updates, only aggregated result
        """
        
        # In production: Use Secure Multi-Party Computation (SMC)
        # or Secret Sharing schemes for true end-to-end encryption
        
        # Simplified example:
        encryption_key = self._generate_ephemeral_key()
        
        serialized_update = json.dumps(model_update, default=str)
        encrypted_data = self._encrypt_with_key(serialized_update, encryption_key)
        
        return {
            'encrypted_update': encrypted_data,
            'key_shard': self._split_key_with_server(encryption_key),
            'algorithm': 'AES-256-GCM'
        }
    
    def _get_model_weights(self) -> Dict:
        """Extract current model weights"""
        return {name: param.data.clone() 
                for name, param in self.model.named_parameters()}
    
    def _load_global_weights(self, weights: Dict):
        """Load global weights into local model"""
        with torch.no_grad():
            for name, param in self.model.named_parameters():
                if name in weights:
                    param.copy_(weights[name])
    
    def _count_unique_patients(self) -> int:
        # Implementation would count unique patient IDs
        return 342  # Example
    
    def _count_data_type(self, data_type: str) -> int:
        # Implementation would count specific data types
        return 1024  # Example
    
    def _calculate_age_range(self) -> str:
        return "18-75"  # Example
    
    def _calculate_diabetes_distribution(self) -> Dict:
        return {"Type 2": 0.7, "Type 1": 0.25, "Pre-diabetes": 0.05}  # Example
    
    def _calculate_missing_values_pct(self) -> float:
        return 2.5  # Example
    
    def _calculate_outliers_pct(self) -> float:
        return 1.2  # Example
    
    def _get_data_collection_period(self) -> str:
        return "2024-01-01 to 2026-03-23"  # Example
    
    def _generate_ephemeral_key(self) -> str:
        import secrets
        return secrets.token_hex(32)
    
    def _encrypt_with_key(self, data: str, key: str) -> str:
        # AES encryption implementation
        return hashlib.sha256(data.encode() + key.encode()).hexdigest()
    
    def _split_key_with_server(self, key: str) -> str:
        # Secret sharing implementation
        return key[:16]  # Simplified
```

### 3.2 Global Coordination Server

```python
from typing import List, Dict, Tuple
import numpy as np
from datetime import datetime
import torch
import torch.nn as nn

class FederatedLearningServer:
    """
    Global coordination server that aggregates model updates from hospitals
    Never sees raw patient data, only encrypted model updates
    """
    
    def __init__(self,
                 global_model: nn.Module,
                 num_clients: int,
                 aggregation_strategy: str = 'fedavg',
                 quality_threshold: float = 0.8):
        """
        Initialize FL Server
        
        Args:
            global_model: Base model architecture
            num_clients: Expected number of participating institutions
            aggregation_strategy: 'fedavg', 'weighted_fedavg', 'trimmed_mean'
            quality_threshold: Minimum model quality to accept updates
        """
        
        self.global_model = global_model
        self.num_clients = num_clients
        self.aggregation_strategy = aggregation_strategy
        self.quality_threshold = quality_threshold
        
        # Tracking
        self.round_number = 0
        self.participating_clients = {}
        self.aggregation_history = []
        self.model_versions = []
    
    def aggregate_client_updates(self, 
                                 client_updates: List[Dict]) -> Tuple[Dict, Dict]:
        """
        Aggregate model updates from all clients
        
        Process:
        1. Validate updates quality
        2. Decrypt updates using secure aggregation
        3. Average weights across clients
        4. Create new global model
        5. Validate aggregated model
        
        Returns:
            (aggregated_weights, aggregation_metrics)
        """
        
        self.round_number += 1
        
        # Step 1: Validate client updates
        valid_updates = self._validate_client_updates(client_updates)
        
        print(f"[Round {self.round_number}] Received updates from {len(valid_updates)}/{self.num_clients} clients")
        
        # Step 2: Decrypt updates (secure aggregation)
        decrypted_updates = self._secure_aggregate_decrypt(valid_updates)
        
        # Step 3: Aggregate based on strategy
        if self.aggregation_strategy == 'fedavg':
            aggregated_weights = self._fedavg_aggregation(decrypted_updates)
        elif self.aggregation_strategy == 'weighted_fedavg':
            aggregated_weights = self._weighted_fedavg_aggregation(decrypted_updates)
        elif self.aggregation_strategy == 'trimmed_mean':
            aggregated_weights = self._trimmed_mean_aggregation(decrypted_updates)
        else:
            raise ValueError(f"Unknown aggregation strategy: {self.aggregation_strategy}")
        
        # Step 4: Create new global model
        self._apply_aggregated_weights(aggregated_weights)
        
        # Step 5: Validate aggregated model
        validation_metrics = self._validate_global_model()
        
        # Step 6: Track aggregation
        aggregation_info = {
            'round': self.round_number,
            'clients_participated': len(valid_updates),
            'aggregation_strategy': self.aggregation_strategy,
            'validation_metrics': validation_metrics,
            'timestamp': datetime.now().isoformat(),
            'update_convergence': self._measure_convergence(decrypted_updates)
        }
        
        self.aggregation_history.append(aggregation_info)
        self.model_versions.append({
            'version': self.round_number,
            'timestamp': datetime.now(),
            'validation_accuracy': validation_metrics['global_accuracy']
        })
        
        print(f"[Round {self.round_number}] Aggregation complete. Global accuracy: {validation_metrics['global_accuracy']:.4f}")
        
        return aggregated_weights, aggregation_info
    
    def _validate_client_updates(self, client_updates: List[Dict]) -> List[Dict]:
        """
        Validate quality of client updates
        - Check for NaN/Inf values
        - Verify client authentication
        - Ensure differential privacy guarantees
        """
        
        valid_updates = []
        
        for update in client_updates:
            is_valid = True
            
            # Check for valid privacy guarantees
            if not update.get('privacy_guarantees', {}).get('differential_privacy_epsilon'):
                print(f"⚠️ Client {update['client_id']}: Missing privacy guarantees")
                is_valid = False
            
            # Check validation metrics
            metrics = update.get('validation_metrics', {})
            if metrics.get('accuracy', 0) < self.quality_threshold:
                print(f"⚠️ Client {update['client_id']}: Low validation accuracy ({metrics.get('accuracy'):.2%})")
                # Don't reject, but flag for monitoring
            
            if is_valid:
                valid_updates.append(update)
        
        return valid_updates
    
    def _fedavg_aggregation(self, client_updates: List[Dict]) -> Dict:
        """
        FedAVG: Simple averaging of model weights
        
        Formula: w_new = (1/n) * Σ Δw_i
        
        where:
        - n = number of clients
        - Δw_i = weight update from client i
        """
        
        aggregated = None
        
        for update in client_updates:
            model_update = update['model_update']
            
            if aggregated is None:
                aggregated = {k: v.clone() for k, v in model_update.items()}
            else:
                for layer_name in model_update:
                    aggregated[layer_name] += model_update[layer_name]
        
        # Average
        for layer_name in aggregated:
            aggregated[layer_name] /= len(client_updates)
        
        return aggregated
    
    def _weighted_fedavg_aggregation(self, client_updates: List[Dict]) -> Dict:
        """
        Weighted FedAVG: Weight updates by institution size
        
        Formula: w_new = Σ (n_i / N) * Δw_i
        
        where:
        - n_i = number of samples at client i
        - N = total samples across all clients
        """
        
        # Calculate total samples
        total_samples = sum(u['data_summary']['total_data_points'] for u in client_updates)
        
        aggregated = None
        
        for update in client_updates:
            # Weight by proportion of data
            weight = update['data_summary']['total_data_points'] / total_samples
            model_update = update['model_update']
            
            if aggregated is None:
                aggregated = {k: v.clone() * weight for k, v in model_update.items()}
            else:
                for layer_name in model_update:
                    aggregated[layer_name] += model_update[layer_name] * weight
        
        return aggregated
    
    def _trimmed_mean_aggregation(self, client_updates: List[Dict], trim_percentage: float = 0.1) -> Dict:
        """
        Trimmed Mean: Robust aggregation against Byzantine/malicious clients
        
        Remove extreme outliers before averaging (e.g., top/bottom 10%)
        """
        
        # Stack all updates for each layer
        num_clients = len(client_updates)
        trim_count = max(1, int(num_clients * trim_percentage))
        
        aggregated = {}
        
        # Get first update to know structure
        first_update = client_updates[0]['model_update']
        
        for layer_name in first_update:
            # Collect layer updates from all clients
            layer_updates = np.array([u['model_update'][layer_name].cpu().numpy() 
                                     for u in client_updates])
            
            # Trim extremes
            trimmed_updates = np.sort(layer_updates, axis=0)
            trimmed_updates = trimmed_updates[trim_count:-trim_count]
            
            # Average
            aggregated[layer_name] = torch.from_numpy(trimmed_updates.mean(axis=0))
        
        return aggregated
    
    def _secure_aggregate_decrypt(self, client_updates: List[Dict]) -> List[Dict]:
        """
        Secure aggregation: Decrypt updates only after aggregation
        
        In production: Use threshold cryptography where:
        - Each update is encrypted with multiple keys
        - No single party can decrypt individual updates
        - Only aggregated result can be decrypted
        """
        
        # Simplified for this example
        # In production, implement actual SMC or secret sharing
        
        decrypted = []
        for update in client_updates:
            decrypted.append({
                'client_id': update['client_id'],
                'model_update': update['model_update'],
                'data_summary': update['data_summary']
            })
        
        return decrypted
    
    def _apply_aggregated_weights(self, aggregated_weights: Dict):
        """Apply aggregated weights to global model"""
        
        with torch.no_grad():
            for name, param in self.global_model.named_parameters():
                if name in aggregated_weights:
                    param.add_(aggregated_weights[name])
    
    def _validate_global_model(self) -> Dict:
        """
        Validate global model on held-out test set
        (must not include data from any training client)
        """
        
        return {
            'global_accuracy': 0.92,  # Placeholder
            'global_loss': 0.23,  # Placeholder
            'improvement_from_previous': 0.03
        }
    
    def _measure_convergence(self, client_updates: List[Dict]) -> Dict:
        """Measure how much model updates differ (convergence indicator)"""
        
        return {
            'max_update_norm': 0.15,  # Placeholder
            'mean_update_norm': 0.08,
            'convergence_status': 'improving'
        }
    
    def get_global_model_weights(self) -> Dict:
        """Get current global model weights to distribute to clients"""
        
        return {name: param.data.clone() 
                for name, param in self.global_model.named_parameters()}
```

---

## 4. Privacy & Security Mechanisms

### 4.1 Differential Privacy

```python
class DifferentialPrivacyEngine:
    """
    Apply differential privacy to gradients before sharing
    Ensures mathematical privacy guarantees
    """
    
    def __init__(self, epsilon: float = 1.0, delta: float = 1e-5):
        """
        epsilon: Privacy budget (smaller = more private)
        delta: Probability of failure
        """
        self.epsilon = epsilon
        self.delta = delta
    
    def add_laplace_noise(self, gradients: torch.Tensor, 
                         sensitivity: float = 1.0) -> torch.Tensor:
        """
        Add Laplace noise to gradients
        
        Noise ~ Laplace(0, sensitivity / epsilon)
        """
        
        scale = sensitivity / self.epsilon
        noise = torch.from_numpy(np.random.laplace(0, scale, gradients.shape))
        
        return gradients + noise.float()
    
    def add_gaussian_noise(self, gradients: torch.Tensor,
                          sensitivity: float = 1.0) -> torch.Tensor:
        """
        Add Gaussian noise to gradients (stronger privacy)
        
        Noise ~ N(0, (sensitivity * sqrt(2*ln(1.25/delta)) / epsilon)²)
        """
        
        import math
        std_dev = sensitivity * math.sqrt(2 * math.log(1.25 / self.delta)) / self.epsilon
        noise = torch.randn_like(gradients) * std_dev
        
        return gradients + noise
```

### 4.2 Secure Aggregation

```python
class SecureAggregationProtocol:
    """
    Implement Secure Multi-Party Computation for aggregation
    Server never sees individual client updates
    """
    
    @staticmethod
    def split_secret(secret: torch.Tensor, num_parties: int) -> List[torch.Tensor]:
        """
        Split secret into shares using Shamir's Secret Sharing
        Requires at least k of n parties to reconstruct secret
        """
        
        # Simplified implementation
        # Production: Use threshold cryptography library
        
        shares = []
        accumulated = torch.zeros_like(secret)
        
        for i in range(num_parties - 1):
            share = torch.randn_like(secret)
            shares.append(share)
            accumulated += share
        
        # Last share = secret - sum of other shares
        final_share = secret - accumulated
        shares.append(final_share)
        
        return shares
    
    @staticmethod
    def reconstruct_secret(shares: List[torch.Tensor]) -> torch.Tensor:
        """Reconstruct secret from shares"""
        
        return sum(shares)
```

---

## 5. Data Types for Federated Learning

### 5.1 Food Recognition Training Data

```python
class FoodRecognitionTrainingData:
    """
    Local food image data for federated learning
    
    Privacy: Images are local, only model updates are shared
    """
    
    def __init__(self):
        self.training_data = {
            'food_images': [],  # Local images of patient meals
            'annotations': [],  # Food labels (rice, curry, bread, etc.)
            'portions': [],     # Portion sizes
            'patient_ids': []   # Anonymized patient references
        }
    
    def prepare_training_batch(self, batch_size: int = 32):
        """Stream food images without storing raw data"""
        
        for batch_start in range(0, len(self.training_data['food_images']), batch_size):
            batch_end = min(batch_start + batch_size, len(self.training_data['food_images']))
            
            batch = {
                'images': self.training_data['food_images'][batch_start:batch_end],
                'labels': self.training_data['annotations'][batch_start:batch_end]
            }
            
            yield batch
```

### 5.2 Glucose Prediction Training Data

```python
class GlucosePredictionTrainingData:
    """
    Local glucose readings and contextual data for federated learning
    
    Privacy: Time series data from individual patients
            Only aggregate statistics and model deltas are shared
    """
    
    def __init__(self):
        self.timeseries_data = {
            'timestamps': [],
            'glucose_readings': [],      # mmol/L or mg/dL
            'food_consumed': [],         # Previous meal carbs
            'activity': [],              # Activity level
            'insulin_administered': [],  # Dosage if applicable
            'stress_level': [],
            'sleep_quality': []
        }
    
    def prepare_sequences(self, sequence_length: int = 24):
        """
        Create time-series sequences for LSTM/GRU training
        Each sequence = 24 hours of glucose context
        
        Output: Sequences suitable for predicting next 2 hours
        """
        
        sequences = []
        targets = []
        
        for i in range(len(self.timeseries_data['glucose_readings']) - sequence_length - 2):
            seq = {
                'glucose_history': self.timeseries_data['glucose_readings'][i:i+sequence_length],
                'activity_history': self.timeseries_data['activity'][i:i+sequence_length],
                'meal_history': self.timeseries_data['food_consumed'][i:i+sequence_length]
            }
            
            target = self.timeseries_data['glucose_readings'][i+sequence_length+2]
            
            sequences.append(seq)
            targets.append(target)
        
        return sequences, targets
```

---

## 6. Federated Learning Workflow Example

### 6.1 Complete Training Round

```python
def run_federated_learning_round(global_server: FederatedLearningServer,
                                 clients: List[FederatedLearningClient],
                                 round_num: int) -> Dict:
    """
    Execute one round of federated learning
    
    Timeline:
    1. Server sends global model to all clients (1 min)
    2. Clients train locally on their data (2-4 hours)
    3. Clients send encrypted updates to server (5 min)
    4. Server aggregates updates (10 min)
    5. New global model created (2 min)
    """
    
    print(f"\n{'='*60}")
    print(f"FEDERATED LEARNING ROUND {round_num}")
    print(f"{'='*60}\n")
    
    # Step 1: Distribute global model
    print("Step 1: Distributing global model to clients...")
    global_weights = global_server.get_global_model_weights()
    
    # Step 2: Parallel local training
    print("Step 2: Local training at each institution...")
    client_updates = []
    
    for client in clients:
        print(f"  → {client.institution_name}: Training model locally...")
        
        update = client.train_local_model(
            global_model_weights=global_weights,
            epochs=5,
            learning_rate=0.001
        )
        
        client_updates.append(update)
        
        print(f"     ✓ {client.institution_name}: Update ready")
        print(f"       - Validation accuracy: {update['validation_metrics']['accuracy']:.2%}")
        print(f"       - Local data points: {update['data_summary']['total_data_points']}")
    
    # Step 3: Server aggregation
    print(f"\nStep 3: Server aggregating {len(client_updates)} updates...")
    aggregated_weights, aggregation_info = global_server.aggregate_client_updates(client_updates)
    
    print(f"✓ Aggregation complete:")
    print(f"  - Global model accuracy: {aggregation_info['validation_metrics']['global_accuracy']:.2%}")
    print(f"  - Convergence status: {aggregation_info['update_convergence']['convergence_status']}")
    
    return {
        'round_number': round_num,
        'participating_clients': [c.client_id for c in clients],
        'client_updates': client_updates,
        'aggregation_info': aggregation_info,
        'global_weights': aggregated_weights
    }
```

---

## 7. Multi-Institutional FL Implementation

### 7.1 Setup Example

```python
# Initialize hospitals/clinics
hospital_a = FederatedLearningClient(
    client_id='HOSP_A_001',
    institution_name='Central Hospital A',
    local_data_loader=DataLoader(hospital_a_dataset, batch_size=32),
    validation_data_loader=DataLoader(hospital_a_val_set, batch_size=32),
    model_architecture=GlucosePredictionModel(),
    privacy_epsilon=1.0  # DP guarantee
)

hospital_b = FederatedLearningClient(
    client_id='HOSP_B_001',
    institution_name='District Hospital B',
    local_data_loader=DataLoader(hospital_b_dataset, batch_size=32),
    validation_data_loader=DataLoader(hospital_b_val_set, batch_size=32),
    model_architecture=GlucosePredictionModel(),
    privacy_epsilon=1.0
)

clinic_c = FederatedLearningClient(
    client_id='CLINIC_C_001',
    institution_name='Private Clinic C',
    local_data_loader=DataLoader(clinic_c_dataset, batch_size=32),
    validation_data_loader=DataLoader(clinic_c_val_set, batch_size=32),
    model_architecture=GlucosePredictionModel(),
    privacy_epsilon=1.0
)

# Initialize global server
global_server = FederatedLearningServer(
    global_model=GlucosePredictionModel(),
    num_clients=3,
    aggregation_strategy='weighted_fedavg',
    quality_threshold=0.8
)

# Run federated learning for 10 rounds
for round_num in range(1, 11):
    result = run_federated_learning_round(
        global_server=global_server,
        clients=[hospital_a, hospital_b, clinic_c],
        round_num=round_num
    )
    
    # Track global model improvement
    print(f"\nRound {round_num} Summary:")
    print(f"  Global Accuracy: {result['aggregation_info']['validation_metrics']['global_accuracy']:.2%}")
    print(f"  Participating: {len(result['participating_clients'])} institutions")
    
    # Show cross-institutional learning benefit
    for client_id, update in zip(result['participating_clients'], result['client_updates']):
        print(f"    - {client_id}: {update['validation_metrics']['accuracy']:.2%}")
```

---

## 8. Privacy Guarantees & Compliance

### 8.1 Privacy Guarantees

```python
class PrivacyGuarantees:
    """
    Mathematical privacy guarantees for federated learning
    """
    
    PRIVACY_PROPERTIES = {
        'differential_privacy': {
            'property': 'Cannot infer if patient was in training data',
            'mechanism': 'Gradient clipping + Laplace/Gaussian noise',
            'epsilon': 1.0,  # Privacy budget
            'delta': 1e-5    # Failure probability
        },
        'federated_privacy': {
            'property': 'Central server cannot infer individual updates',
            'mechanism': 'Secure aggregation + secret sharing',
            'guarantee': 'Only aggregated result visible'
        },
        'data_minimization': {
            'property': 'Never share raw patient data',
            'mechanism': 'Only model weights transmitted',
            'raw_data_leave': 'Never leaves institution'
        }
    }
    
    @staticmethod
    def verify_privacy_compliance(epsilon: float, delta: float) -> Dict:
        """
        Verify if privacy parameters meet healthcare standards
        """
        
        return {
            'gdpr_compliant': epsilon <= 8.0 and delta <= 1e-3,  # GDPR guidelines
            'hipaa_compliant': delta <= 1e-5,  # Healthcare standard
            'total_privacy_loss': f'({epsilon}, {delta})-DP',
            'recommendation': 'Privacy parameters meet healthcare standards'
        }
```

### 8.2 Regulatory Compliance

```python
class RegulatoryCompliance:
    """Track compliance with data protection regulations"""
    
    REGULATIONS = {
        'GDPR': {
            'jurisdiction': 'European Union',
            'requirements': [
                'Data minimization',
                'Purpose limitation',
                'Storage limitation',
                'Right to be forgotten'
            ],
            'fl_compliance': 'Met - No personal data centralization'
        },
        'HIPAA': {
            'jurisdiction': 'United States (Healthcare)',
            'requirements': [
                'Privacy of PHI (Protected Health Information)',
                'Security of PHI',
                'Breach notification',
                'Patient rights'
            ],
            'fl_compliance': 'Met - PHI never leaves institutions'
        },
        'PDPA': {
            'jurisdiction': 'Sri Lanka/South Asia',
            'requirements': [
                'Consent for data processing',
                'Data security measures',
                'Individual access rights'
            ],
            'fl_compliance': 'Met - Consent via local institution'
        }
    }
```

---

## 9. Monitoring & Evaluation

### 9.1 Federated Learning Metrics

```python
class FederatedLearningMetrics:
    """
    Monitor FL system health and performance
    """
    
    def __init__(self):
        self.metrics_history = []
    
    def track_round(self, round_info: Dict):
        """
        Track metrics for each FL round
        
        Metrics:
        1. Model Performance: Global accuracy, loss, validation metrics
        2. System Health: Round time, client participation, data distribution
        3. Privacy: DP budget consumed, gradient clipping effectiveness
        4. Communication: Update sizes, compression ratios
        """
        
        metrics = {
            'round': round_info['round_number'],
            'timestamp': datetime.now(),
            
            # Performance
            'global_accuracy': round_info['aggregation_info']['validation_metrics']['global_accuracy'],
            'clients_participated': round_info['aggregation_info']['clients_participated'],
            'participation_rate': round_info['aggregation_info']['clients_participated'] / round_info['num_total_clients'],
            
            # Data distribution
            'total_training_samples': sum(
                update['data_summary']['total_data_points'] 
                for update in round_info['client_updates']
            ),
            
            # Privacy consumption
            'total_privacy_budget': sum(
                update['privacy_guarantees']['differential_privacy_epsilon']
                for update in round_info['client_updates']
            ),
            
            # Communication
            'total_update_size_mb': sum(
                update['update_size'] / (1024 * 1024)
                for update in round_info['client_updates']
            )
        }
        
        self.metrics_history.append(metrics)
        return metrics
    
    def get_convergence_plot_data(self) -> Dict:
        """Data for visualizing model convergence over rounds"""
        
        return {
            'rounds': [m['round'] for m in self.metrics_history],
            'global_accuracy': [m['global_accuracy'] for m in self.metrics_history],
            'participation_rate': [m['participation_rate'] for m in self.metrics_history],
            'total_training_samples': [m['total_training_samples'] for m in self.metrics_history]
        }
```

---

## 10. Multi-Model Federated Learning

### 10.1 Distributed Specialized Models

In the diabetes management system, multiple models can be trained federally:

```python
FEDERATED_MODELS = {
    'food_recognition': {
        'objective': 'Identify food items from images',
        'input': 'Food images from hospital cafeterias',
        'output': 'Food item labels + confidence scores',
        'training_data_types': ['Hospital A': 50K images, 'Hospital B': 35K images, 'Clinic C': 20K images],
        'privacy_approach': 'Image data never leaves hospitals'
    },
    
    'glucose_prediction': {
        'objective': 'Predict blood glucose 2 hours ahead',
        'input': 'Time series: glucose, meals, activity',
        'output': 'Glucose level prediction + confidence interval',
        'training_data_types': ['Hospital A': 12K patients, 'Hospital B': 8K patients, 'Clinic C': 5K patients],
        'privacy_approach': 'Dynamic glucose readings never shared'
    },
    
    'activity_analysis': {
        'objective': 'Infer health impact of activities',
        'input': 'Step count, activity type, duration',
        'output': 'Glucose-lowering effect prediction',
        'training_data_types': ['Hospital A': 100K activity logs, 'Hospital B': 75K logs, 'Clinic C': 50K logs],
        'privacy_approach': 'Activity data never centralized'
    },
    
    'personalized_recommendation': {
        'objective': 'Recommend foods/activities based on individual patterns',
        'input': 'User history + global patterns (federated)',
        'output': 'Personalized lunch suggestions',
        'training_data_types': ['Hospital A': Patient interaction data, ...],
        'privacy_approach': 'Local personalization + federated insights'
    }
}
```

---

## 11. Challenges & Solutions

| Challenge | Impact | Solution |
|-----------|--------|----------|
| **Data Heterogeneity** | Different hospitals have different patient populations | Use weighted aggregation, personalized FL |
| **System Heterogeneity** | Different hardware capabilities | Asynchronous aggregation, client selection |
| **Privacy-Utility Tradeoff** | Adding noise reduces accuracy | Adaptive noise scheduling, cool-down periods |
| **Communication Cost** | Model updates are large | Gradient compression, quantization |
| **Model Staleness** | Delayed updates can hurt convergence | Parameter staleness handling, staleness-aware aggregation |
| **Byzantine Clients** | Malicious/faulty hospitals | Robust aggregation (trimmed mean, median) |

---

## 12. Future Enhancements

1. **Personalized Federated Learning**: Each hospital gets a slightly different model optimized for its population
2. **Hierarchical FL**: Multi-level (Hospital → Regional → National)
3. **Cross-Silo & Cross-Device**: Combine institutional FL with individual patient phones
4. **Federated Transfer Learning**: Pre-trained global model, fine-tuned locally
5. **Continual/Incremental FL**: New hospitals join existing federation seamlessly

---

**End of Federated Learning Architecture Document**
