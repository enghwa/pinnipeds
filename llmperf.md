


```bash
git clone https://github.com/ray-project/llmperf.git
cd llmperf
docker build -t llmperf .
```
dockerfile for `public.ecr.aws/u3r1l1j7/eks-genai:llmperf`
```
FROM python:3.10.14-slim-bullseye
ADD . /a/
RUN cd /a && pip install -e .

```

example of llmperf call
```
python token_benchmark_ray.py \
--model "meta-llama/Meta-Llama-3-8B" \
--mean-input-tokens 550 \
--stddev-input-tokens 150 \
--mean-output-tokens 150 \
--stddev-output-tokens 10 \
--max-num-completed-requests 1 \
--timeout 600 \
--num-concurrent-requests 1 \
--results-dir "result_outputs" \
--llm-api openai \
--additional-sampling-params '{}'
```


**k8s job**

```
apiVersion: batch/v1
kind: Job
metadata:
  name: llmperf-job
spec:
  template:
    spec:
      containers:
      - name: llmperf-job
        image: public.ecr.aws/u3r1l1j7/eks-genai:llmperf
        resources:
          requests:
            memory: "20Gi"  
        command:
        - "/bin/sh"
        - "-c"
        - "python /a/token_benchmark_ray.py --model \"$model\" --mean-input-tokens 1024 --stddev-input-tokens 0 --mean-output-tokens 1024 --stddev-output-tokens 100 --max-num-completed-requests 1 --timeout 600 --num-concurrent-requests 1 --results-dir \"result_outputs\" --llm-api openai --additional-sampling-params '{}'"
        env:
        - name: OPENAI_API_KEY
          value: "XXXX"
        - name: OPENAI_API_BASE
          value: "http://xxxxxxxxxxxxxx/v1"
        - name: model
          value: "meta-llama/Meta-Llama-3-8B"
      restartPolicy: Never
  backoffLimit: 1
```
