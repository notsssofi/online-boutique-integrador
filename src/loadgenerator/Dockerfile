# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed unde the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM python:3.12.8-alpine

RUN apk update && apk add --no-cache \
    g++ \
    make \
    libstdc++ \
    linux-headers \
    wget \
    && rm -rf /var/cache/apk/*

WORKDIR /loadgen

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

ENTRYPOINT ["locust", "-f", "loadgen.py"]

