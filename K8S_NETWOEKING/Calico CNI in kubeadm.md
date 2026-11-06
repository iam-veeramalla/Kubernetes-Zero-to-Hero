First, why do we need Calico?

* Imagine you created a Kubernetes cluster using kubeadm:

      - One control-plane node.
      - Two worker nodes.
      - Multiple Pods running on different nodes.

<img width="760" height="261" alt="image" src="https://github.com/user-attachments/assets/bf6e9af6-81fd-4fa9-867e-757746ef203e" />

How will the frontend Pod on Worker 1 communicate with the database Pod on Worker 2?

* Kubernetes needs a networking solution to provide Pod-to-Pod connectivity.

* That's where Calico comes in.

* Calico provides Pod networking and can enforce network policies.

<img width="812" height="341" alt="image" src="https://github.com/user-attachments/assets/a1210ef4-0632-4d7b-bedc-d7870bf9d8f8" />

How does Calico work when a Pod is created?
---------------------------------------------

* Let's understand the actual flow.

<img width="927" height="705" alt="image" src="https://github.com/user-attachments/assets/e695e132-5d5d-4e3a-a40d-e6fb7121ebf8" />

<img width="537" height="316" alt="image" src="https://github.com/user-attachments/assets/4b56f4f5-89d1-4bb8-9fe9-aa7e220c74f7" />

How does Pod-to-Pod communication work across nodes?
---------------------------------------------------

<img width="952" height="528" alt="image" src="https://github.com/user-attachments/assets/ba8568cf-9ccf-4115-ae6f-6d8462ccd799" />

<img width="525" height="535" alt="image" src="https://github.com/user-attachments/assets/d18e64da-4b36-4201-bf30-61ee7705d0ec" />

How does traffic reach the other node?
---------------------------------------

Calico supports different networking modes. Let's understand the two important ones.

<img width="708" height="301" alt="image" src="https://github.com/user-attachments/assets/0609ae55-c396-4673-bd89-b615a0186cd9" />
<img width="961" height="662" alt="image" src="https://github.com/user-attachments/assets/cba263c8-4157-49c0-8e19-37260352cdd0" />

<img width="937" height="390" alt="image" src="https://github.com/user-attachments/assets/7dfe488c-d683-4275-87a8-766a62d4f84b" />

<img width="767" height="536" alt="image" src="https://github.com/user-attachments/assets/ac531a9e-50b5-4746-8f59-4324d67be77e" />
