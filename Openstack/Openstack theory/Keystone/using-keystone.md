
# Sử dụng keystone

[I. Sử dụng command line để thực hiện những tác vụ quản trị cơ bản trong Keystone](#command)

- [1. Lấy Token](#token)
- [2. Liệt kê danh sách users](#user)
- [3. Liệt kê danh sách projects](#project)
- [4. Liệt kê danh sách groups](#group)
- [5. Liệt kê danh sách roles](#role)
- [6. Liệt kê danh sách domains](#domain-list)
- [7. Tạo domains](#domain-create)
- [8. Tạo project với domain](#create-project)
- [9. Tạo user với domain](#create-user)
- [10. Gán role cho user vào project](#add-user)
- [11. Xác thực user mới](#auth-user)

[II. Sử dụng dashboard để thực hiện những tác vụ quản trị cơ bản trong Keystone](#dashboard)

- [1. Những tác vụ có thể thực hiện được của Keystone thông qua dashboard](#task)
- [2. Liệt kê, thiết lập, xóa, tạo, xem project](#project-dashboard)
- [3. Liệt kê, thiết lập, xóa, tạo, xem user](#user-dashboard)

----------

### <a name="command"> I. Sử dụng command line để thực hiện những tác vụ quản trị cơ bản trong Keystone </a>

Tại phần này chúng ta sẽ sử dụng OpenStackClient để thực hiện những tác vụ quản trị cơ bản với Keystone như xác thực, liệt kê danh sách, tạo mới users, domains, projects,...

Để bắt đầu, ta cần phải thiết lập 1 số biến môi trường (environment variables) để không phải lặp lại nhiều lần trong quá trình xác thực.

Chạy những câu lệnh sau:

``` sh
export OS_IDENTITY_API_VERSION=3
export OS_AUTH_URL=http://controller:5000/v3
export OS_USERNAME=admin
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PASSWORD=ducnm37
export OS_PROJECT_DOMAIN_NAME=Default
```

Lưu ý: Hầu hết những thông tin phía trên nên trùng với mô hình của riêng bạn ngoại trừ hostname hoặc địa chỉ ip của máy ảo.

Để kiểm tra xem biết đã được set hay chưa, chạy lệnh sau:

env | grep OS

```
OS_IDENTITY_API_VERSION=3
OS_AUTH_URL=http://controller:5000/v3
OS_USERNAME=admin
OS_PROJECT_NAME=admin
OS_USER_DOMAIN_NAME=Default
OS_PASSWORD=ducnm37
OS_PROJECT_DOMAIN_NAME=Default
```

#### <a name="token"> 1. Lấy Token </a>

**Dùng OpenStackClient**

Vì đã thực thi việc xác thực và trao quyền bằng các biến môi trường, bạn chỉ cần chạy câu lệnh sau để lấy token:

``` sh
[root@controller ~]# openstack token issue
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| expires    | 2017-06-07T09:34:30.284479Z      |
| id         | 5548f103dc474b88903fcd67ddc4d4c0 |
| project_id | b47e8a9d5d88439dadc4f849c0424e8c |
| user_id    | cd171df00daf44208ba66c8fab9595b9 |
+------------+----------------------------------+
```

**Dùng cURL**

Khi dùng cURL, phần payload phải chứa thông tin về user và project.

``` sh
$ curl -i -H "Content-Type: application/json" -d '
{ "auth": {
    "identity": {
        "methods": ["password"],
        "password": {
            "user": {
              "name": "admin",
              "domain": { "name": "Default" },
              "password": "ducnm37"
            }
          }
        },
        "scope": {
          "project": {
            "name": "admin",
            "domain": { "name": "Default" }
          }
        }
      }
}' http://localhost:5000/v3/auth/tokens
```

- Phần response:

```sh
HTTP/1.1 201 Created
Date: Tue, 14 Aug 2018 10:38:14 GMT
Server: Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5
X-Subject-Token: gAAAAABbcrEXB9u3UFLtWqLnw8QV3NLyQ8BdADmO-HQrCq5WUbYv8yH9q32MBVKE52A5b92HKCYX81GMQSV2hlPvO-iZivFlTx2mVyWT3REJFrdyRSX_N-0xnrTfuAS78FN0Ktp0xWDl8J_cndghoeBs8hCsX95ObULRBLKuHTbECNjF6e0S77U
Vary: X-Auth-Token
x-openstack-request-id: req-a4eb2f43-4f40-432c-94d6-60864a999aa1
Content-Length: 4761
Content-Type: application/json


{"token": {"methods": ["password"], "roles": [{"id": "2c43533ecb594bde953f7ad2d7ee4a31", "name": "admin"}], "expires_at": "2017-06-07T09:38:01.780859Z", "project": {"domain": {"id": "default", "name": "Default"}, "id": "b47e8a9d5d88439dadc4f849c0424e8c", "name": "admin"}, "catalog": [{"endpoints": [{"region_id": "RegionOne", "url": "http://192.168.100.171:35357/v2.0", "region": "RegionOne", "interface": "admin", "id": "6f325fa91ef14b25baf9251bea74e121"}, {"region_id": "RegionOne", "url": "http://192.168.100.171:5000/v2.0", "region": "RegionOne"
```

Phần response phía trên đã được rút ngắn lại cho dễ đọc bởi nó chứa catalog với tất cả các endpoints nên khá dài.

Token nằm ở phía sau `X-Subject-Token`. Hãy set nó vào biến để không phải khai báo ở những thao tác tiếp theo:

`OS_TOKEN=gAAAAABbcrEXB9u3UFLtWqLnw8QV3NLyQ8BdADmO-HQrCq5WUbYv8yH9q32MBVKE52A5b92HKCYX81GMQSV2hlPvO-iZivFlTx2mVyWT3REJFrdyRSX_N-0xnrTfuAS78FN0Ktp0xWDl8J_cndghoeBs8hCsX95ObULRBLKuHTbECNjF6e0S77U`

#### <a name="user"> 2. Liệt kê danh sách users </a>

**Sử dụng OpenStackClient**

Sử dụng câu lệnh sau để xem danh sách users:

``` sh
[root@controller ~]# openstack user list
+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
| 7d28b37e3cff4540862e10be712c6c93 | demo      |
| 95790ba9669a41e4ac8c374860a76de6 | admin     |
| 99823ae352704742afd80f4d443ca17b | placement |
| 9bd13f620ccf4d8a89ed16214b2341e9 | cinder    |
| a63a966b07f248c6a05f4faf5c221185 | nova      |
| c5a4909aecea45118fad6d36b2de8a90 | neutron   |
| e382a94fcd1043988bfa5e8e208f8036 | glance    |
+----------------------------------+-----------+

```

**Sử dụng cURL**

Để sử dụng cURL, bạn phải khai báo token và địa chỉ API tương ứng

``` sh
curl -s -H "X-Auth-Token: $OS_TOKEN" \
 http://localhost:5000/v3/users | python -mjson.tool
```

Kết quả:

``` sh
{
    "links": {
        "next": null,
        "previous": null,
        "self": "http://localhost:5000/v3/users"
    },
    "users": [
        {
            "domain_id": "default",
            "enabled": true,
            "id": "7d28b37e3cff4540862e10be712c6c93",
            "links": {
                "self": "http://localhost:5000/v3/users/7d28b37e3cff4540862e10be712c6c93"
            },
            "name": "demo",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "95790ba9669a41e4ac8c374860a76de6",
            "links": {
                "self": "http://localhost:5000/v3/users/95790ba9669a41e4ac8c374860a76de6"
            },
            "name": "admin",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "99823ae352704742afd80f4d443ca17b",
            "links": {
                "self": "http://localhost:5000/v3/users/99823ae352704742afd80f4d443ca17b"
            },
            "name": "placement",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "9bd13f620ccf4d8a89ed16214b2341e9",
            "links": {
                "self": "http://localhost:5000/v3/users/9bd13f620ccf4d8a89ed16214b2341e9"
            },
            "name": "cinder",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "a63a966b07f248c6a05f4faf5c221185",
            "links": {
                "self": "http://localhost:5000/v3/users/a63a966b07f248c6a05f4faf5c221185"
            },
            "name": "nova",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "c5a4909aecea45118fad6d36b2de8a90",
            "links": {
                "self": "http://localhost:5000/v3/users/c5a4909aecea45118fad6d36b2de8a90"
            },
            "name": "neutron",
            "options": {},
            "password_expires_at": null
        },
        {
            "domain_id": "default",
            "enabled": true,
            "id": "e382a94fcd1043988bfa5e8e208f8036",
            "links": {
                "self": "http://localhost:5000/v3/users/e382a94fcd1043988bfa5e8e208f8036"
            },
            "name": "glance",
            "options": {},
            "password_expires_at": null
        }
    ]
}

}
```


#### <a name="project"> 3. Liệt kê danh sách projects </a>

**Sử dụng OpenStackClient**

Sử dụng câu lệnh sau:

```sh
[root@controller ~]# openstack project list
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| 0bbb715e7e81497494efa275a6291732 | service |
| 85330b6a2cda4e88b39f01bc688f8b8f | demo    |
| 961859bc0b594d0491e77c285b376851 | admin   |
+----------------------------------+---------+

```

**Sử dụng cURL**

Dùng cURL với endpoints của projects

```sh
curl -s -H "X-Auth-Token: $OS_TOKEN" \
 http://localhost:5000/v3/projects | python -mjson.tool
```

Kết quả:

``` sh
> http://localhost:5000/v3/projects | python -mjson.tool
{
    "links": {
        "next": null,
        "previous": null,
        "self": "http://localhost:5000/v3/projects"
    },
    "projects": [
        {
            "description": "Service Project",
            "domain_id": "default",
            "enabled": true,
            "id": "0bbb715e7e81497494efa275a6291732",
            "is_domain": false,
            "links": {
                "self": "http://localhost:5000/v3/projects/0bbb715e7e81497494efa275a6291732"
            },
            "name": "service",
            "parent_id": "default",
            "tags": []
        },
        {
            "description": "Demo Project",
            "domain_id": "default",
            "enabled": true,
            "id": "85330b6a2cda4e88b39f01bc688f8b8f",
            "is_domain": false,
            "links": {
                "self": "http://localhost:5000/v3/projects/85330b6a2cda4e88b39f01bc688f8b8f"
            },
            "name": "demo",
            "parent_id": "default",
            "tags": []
        },
        {
            "description": "Bootstrap project for initializing the cloud.",
            "domain_id": "default",
            "enabled": true,
            "id": "961859bc0b594d0491e77c285b376851",
            "is_domain": false,
            "links": {
                "self": "http://localhost:5000/v3/projects/961859bc0b594d0491e77c285b376851"
            },
            "name": "admin",
            "parent_id": "default",
            "tags": []
        }
    ]
}

```

#### <a name="group"> 4. Liệt kê danh sách groups </a>

**Sử dụng OpenStackClient**

Dùng câu lệnh sau:

``` sh
[root@controller ~]# openstack group list
```

Lưu ý: Nếu không có group thì sẽ không có giá trị nào trả về

**Sử dụng cURL**

``` sh
curl -s -H "X-Auth-Token: $OS_TOKEN" \
 http://localhost:5000/v3/groups | python -mjson.tool
```

Kết quả:

``` sh
{
    "groups": [],
    "links": {
        "next": null,
        "previous": null,
        "self": "http://localhost:5000/v3/groups"
    }
}

```

#### <a name="role"> 5. Liệt kê danh sách roles </a>

**Sử dụng OpenStackClient**

Sử dụng câu lệnh sau:

``` sh
[root@controller ~]# openstack role list
+----------------------------------+-------+
| ID                               | Name  |
+----------------------------------+-------+
| 737174ad500242e18d9167c9e1448c8a | user  |
| dc71394d5d674fefb9e321b9d9cc4857 | admin |
+----------------------------------+-------+
```

**Sử dụng cURL**

``` sh
curl -s -H "X-Auth-Token: $OS_TOKEN" \
 http://localhost:5000/v3/roles | python -mjson.tool
```

Kết quả:

``` sh
{
    "links": {
        "next": null,
        "previous": null,
        "self": "http://localhost:5000/v3/roles"
    },
    "roles": [
        {
            "domain_id": null,
            "id": "737174ad500242e18d9167c9e1448c8a",
            "links": {
                "self": "http://localhost:5000/v3/roles/737174ad500242e18d9167c9e1448c8a"
            },
            "name": "user"
        },
        {
            "domain_id": null,
            "id": "dc71394d5d674fefb9e321b9d9cc4857",
            "links": {
                "self": "http://localhost:5000/v3/roles/dc71394d5d674fefb9e321b9d9cc4857"
            },
            "name": "admin"
        }
    ]
}
```

#### <a name="domain-list"> 6. Liệt kê danh sách domains </a>

**Sử dụng OpenStackClient**

Sử dụng câu lệnh sau:

``` sh
[root@controller ~]# openstack domain list
+---------+---------+---------+--------------------+
| ID      | Name    | Enabled | Description        |
+---------+---------+---------+--------------------+
| default | Default | True    | The default domain |
+---------+---------+---------+--------------------+
```

**Sử dụng cURL**

``` sh
curl -s -H "X-Auth-Token: $OS_TOKEN" \
 http://localhost:5000/v3/domains | python -mjson.tool
```

Kết quả:

``` sh
{
    "domains": [
        {
            "description": "The default domain",
            "enabled": true,
            "id": "default",
            "links": {
                "self": "http://localhost:5000/v3/domains/default"
            },
            "name": "Default",
            "tags": []
        }
    ],
    "links": {
        "next": null,
        "previous": null,
        "self": "http://localhost:5000/v3/domains"
    }
}

```

#### <a name="domain-create"> 7. Tạo domains </a>

**Sử dụng OpenStackClient**

Sử dụng câu lệnh sau để tạo domain `acme`

``` sh
[root@controller ~]# openstack domain create acme
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description |                                  |
| enabled     | True                             |
| id          | 3f179fc827bf4f438c4612d84438c1ce |
| name        | acme                             |
| tags        | []                               |
+-------------+----------------------------------+

```

**Sử dụng cURL**

Chúng ta sử dụng POST request với một chút thông tin ở phần payload (tên domain) và mặc định nó sẽ được bật sau khi tạo xong

``` sh
[root@controller ~]# curl -s -H "X-Auth-Token: $OS_TOKEN" -H "Content-Type: application/json" -d '{ "domain": { "name": "acme2"}}' http://localhost:5000/v3/domains | python -mjson.tool

{
    "domain": {
        "description": "",
        "enabled": true,
        "id": "423c901f4fdf403aa8950872bac6752e",
        "links": {
            "self": "http://localhost:5000/v3/domains/423c901f4fdf403aa8950872bac6752e"
        },
        "name": "acme2",
        "tags": []
    }
}

```

#### <a name="create-project"> 8. Tạo project với domain </a>

**Sử dụng OpenStackClient**

Ta cần thêm một vài mô tả để có thể tạo project với domain

``` sh
[root@controller ~]# openstack project create ducnm_project --domain acme --description "ducnm project"
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | ducnm project                    |
| domain_id   | 3f179fc827bf4f438c4612d84438c1ce |
| enabled     | True                             |
| id          | 91a403a2dd574c1f9b26e928789afe70 |
| is_domain   | False                            |
| name        | ducnm_project                    |
| parent_id   | 3f179fc827bf4f438c4612d84438c1ce |
| tags        | []                               |
+-------------+----------------------------------+
```

**Sử dụng cURL**

Cùng với tên của project, phần payload còn cần phải có domain ID mà ta có ở phần tạo domain trước đó:

``` sh
curl -s -H "X-Auth-Token: $OS_TOKEN" -H "Content-Type: application/json" -d '{ "project": { "name": "tims_project 2", "domain_id": "3f179fc827bf4f438c4612d84438c1ce", "description": "tims dev project 2"}}' http://localhost:5000/v3/projects | python -mjson.tool

{
    "project": {
        "description": "tims dev project 2",
        "domain_id": "3f179fc827bf4f438c4612d84438c1ce",
        "enabled": true,
        "id": "f32194ede1934fce93b1b14a2fd3cd49",
        "is_domain": false,
        "links": {
            "self": "http://localhost:5000/v3/projects/f32194ede1934fce93b1b14a2fd3cd49"
        },
        "name": "tims_project 2",
        "parent_id": "3f179fc827bf4f438c4612d84438c1ce",
        "tags": []
    }
}

```

#### <a name="create-user"> 9. Tạo user với domain </a>

**Sử dụng OpenStackClient**

Ta cần cung cấp thông tin của domain. Password và email cho user là optional.

``` sh
[root@controller ~]# openstack user create ducnm --email ducnm37@fpt.com.vn \
>  --domain acme --description "ducnm openstack user account" \
>  --password 123456
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| description         | ducnm openstack user account     |
| domain_id           | 3f179fc827bf4f438c4612d84438c1ce |
| email               | ducnm37@fpt.com.vn               |
| enabled             | True                             |
| id                  | 76032524c4034c179848e41ed570861e |
| name                | ducnm                            |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

```

**Sử dụng cURL**

Tương tự như những trường hợp trước, ta cũng cần khai báo đúng cấu trúc trong payload:

``` sh
[root@controller ~]#  curl -s -H "X-Auth-Token: $OS_TOKEN" -H "Content-Type: application/json" -d '{ "user": { "name": "ducnm37", "password": "123456", "email": "ducnm37@fpt.com.vn", "domain_id": "3f179fc827bf4f438c4612d84438c1ce", "description": "ducnm openstack user account"}}' http://localhost:5000/v3/users | python -mjson.tool
 {
    "user": {
        "description": "ducnm user account",
        "domain_id": "3f179fc827bf4f438c4612d84438c1ce",
        "email": "ducnm37@fpt.com.vn",
        "enabled": true,
        "id": "4cca697f7b414cc19bb48b46195f5b71",
        "links": {
            "self": "http://localhost:5000/v3/users/4cca697f7b414cc19bb48b46195f5b71"
        },
        "name": "ducnm37",
        "options": {},
        "password_expires_at": null
    }
}

```

#### <a name="add-user"> 10. Gán role cho user vào project

**Sử dụng OpenStackClient**

Để gán role cho user mới, ta có thể sử dụng command line. Bạn cần khai báo đúng project, user và domain, nếu không OpenStackClient sẽ dùng domain mặc định.

``` sh
openstack role add admin --project ducnm_project --project-domain acme --user ducnm --user-domain acme
```

**Sử dụng cURL**

Khác với những command trước, nó sử dụng PUT thay vì POST và chỉ dùng id của user, project và role.

``` sh
curl -s -X PUT -H "X-Auth-Token: $OS_TOKEN" \
http://localhost:5000/v3/projects/91a403a2dd574c1f9b26e928789afe70/users/76032524c4034c179848e41ed570861e/roles/dc71394d5d674fefb9e321b9d9cc4857
```

#### <a name="auth-user"> 11. Xác thực user mới

**Sử dụng OpenStackClient**

Để xác thực user mới, tốt nhất ta nên tạo mới 1 session và thay đổi thiết lập biến môi trường tùy thuộc theo user mới:

``` sh
export OS_PASSWORD=123456
export OS_IDENTITY_API_VERSION=3
export OS_AUTH_URL=http://controller:5000/v3
export OS_USERNAME=ducnm
export OS_PROJECT_NAME=ducnm_project
export OS_USER_DOMAIN_NAME=acme
export OS_PROJECT_DOMAIN_NAME=acme
```

Sau khi thiết lập ta có thể lấy token và dĩ nhiên user đã được xác thực:

``` sh
[root@controller ~]# openstack token issue
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2018-08-14T11:58:16+0000                                                                                                                                                                |
| id         | gAAAAABbcrXIyRETBMrjo4w1iwvul-Ta42mDALLuqnW3YhtRr0vOBh-2RpnlNLck24KC5bhcP4kMCFY0-DPU9faOD-iNZPHApDi1psW3SL2seqHSNGw07OhNKqea9dlfphDjqLWq6XbFFBXA_ZZhgDMthCeQbLw3ME99bpkD3a1hWQq4YBxO0r0 |
| project_id | 91a403a2dd574c1f9b26e928789afe70                                                                                                                                                        |
| user_id    | 76032524c4034c179848e41ed570861e                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

**Sử dụng cURL**

Giống với phần trước đó khi chúng ta lấy token cho admin tuy nhiên có một vài giá trị cần thay đổi cho phù hợp với user mới.

``` sh
curl -i -H "Content-Type: application/json" -d ' { "auth": {
  "identity": {
    "methods": ["password"],
    "password": {
      "user": {
        "name": "ducnm",
        "domain": { "name": "acme" },
        "password": "123456"
      }
    }
  },
  "scope": {
    "project": {
      "name": "ducnm_project",
      "domain": { "name": "acme" }
      }
    }
  }
}' http://localhost:5000/v3/auth/tokens
```

Kết quả:

``` sh
HTTP/1.1 201 Created
Date: Tue, 14 Aug 2018 10:59:01 GMT
Server: Apache/2.4.6 (CentOS) mod_wsgi/3.4 Python/2.7.5
X-Subject-Token: gAAAAABbcrX1pmnYDXVz5UNuNmXhhTOUy0L0I9LBxzs57L9T5DskQJluluO9WNOPq_fAwoqOM03K2cathHZCugBMeGsZfajF_VwffXqEwwsE42rdddRs0ah4MMa0Ogq72PPOIOsLJN3g75MpGm9W0ZEEgJVeR4-kHS2nAbJc_bns1KSNZ5ke004
Vary: X-Auth-Token
x-openstack-request-id: req-b08ea7a4-2968-4531-984a-4e9a2048cc45
Content-Length: 4813
Content-Type: application/json

```

### <a name="dashboard"> II. Sử dụng dashboard để thực hiện những tác vụ quản trị cơ bản trong Keystone

#### <a name="task"> 1. Những tác vụ có thể thực hiện được của Keystone thông qua dashboard

Tùy thuộc vào phiên bản được sử dụng trong file config của Horizon. Nếu là Identity API v2 thì chỉ có phần quản lí cho User và Project. Nếu là v3 thì sẽ có thêm Group, Domain, và Role.


#### <a name="project-dashboard"> 2. Liệt kê, thiết lập, xóa, tạo, xem project

<img src="https://i.imgur.com/PI1MZIB.png">

#### <a name="user-dashboard"> 3. Liệt kê, thiết lập, xóa, tạo, xem user

<img src="https://i.imgur.com/i1vrc8Z.png">
