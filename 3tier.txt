provider "aws" {
    region = "ap-south-1"
    profile = "aradhya"
  }
resource "aws_vpc" "ttier" {
  cidr_block = "10.0.0.0/16"

}

resource "aws_subnet" "public" {
  vpc_id            = aws_vpc.ttier.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "ap-south-1a"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "private1" {
  vpc_id            = aws_vpc.ttier.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "ap-south-1b"
  map_public_ip_on_launch = false
}

resource "aws_subnet" "private2" {
  vpc_id            = aws_vpc.ttier.id
  cidr_block        = "10.0.3.0/24"
  availability_zone = "ap-south-1c"
  map_public_ip_on_launch = false

}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.ttier.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.internet.id
  }

  tags = {
    Name = "Public Route Table"
  }
}
resource "aws_internet_gateway" "internet" {
    vpc_id = aws_vpc.ttier.id
    tags = {
      Name: "igw_vpc"
    }
}
resource "aws_route_table_association" "public_subnet_association" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

resource "aws_security_group" "public_sg" {
  name        = "public_sg"
  description = "Security group for public instance"
  vpc_id      = aws_vpc.ttier.id
    ingress {
        from_port = 0
        to_port = 0
        protocol = "-1"    
        cidr_blocks = ["0.0.0.0/0"] 
    }
    egress {
        from_port = 0
        to_port = 0
        protocol = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }

}

resource "aws_security_group" "private1_sg" {
  name        = "private1_sg"
  description = "Security group for private instance 1"
  vpc_id      = aws_vpc.ttier.id
      ingress {
        from_port = 0
        to_port = 0
        protocol = "-1"    
        cidr_blocks = ["0.0.0.0/0"] 
    }
    egress {
        from_port = 0
        to_port = 0
        protocol = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }


}

resource "aws_security_group" "private2_sg" {
  name        = "private2_sg"
  description = "Security group for private instance 2"
  vpc_id      = aws_vpc.ttier.id
      ingress {
        from_port = 0
        to_port = 0
        protocol = "-1"    
        cidr_blocks = ["0.0.0.0/0"] 
    }
    egress {
        from_port = 0
        to_port = 0
        protocol = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }


}

resource "aws_instance" "public_instance" {
  ami           = "ami-01376101673c89611"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id
  security_groups = [aws_security_group.public_sg.id]
  tags = {
    Name = "Nginx"
  }

}

resource "aws_instance" "private_instance1" {
  ami           = "ami-01376101673c89611"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.private1.id
  security_groups = [aws_security_group.private1_sg.id]
  tags = {
    Name = "Apache Tomcat"
  }


}

resource "aws_instance" "private_instance2" {
  ami           = "ami-01376101673c89611"
  instance_type = "t3a.micro"
  subnet_id     = aws_subnet.private2.id
  security_groups = [aws_security_group.private2_sg.id]
    tags = {
    Name = "Database"
  }


}

resource "aws_db_instance" "main" {
  allocated_storage    = 10
  db_name              = "mydb"
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  username             = "aradhya"
  password             = "admin123"
  parameter_group_name = "default.mysql8.0"
  skip_final_snapshot  = true
}

resource "aws_db_subnet_group" "main" {
  name       = "main"
  subnet_ids = [aws_subnet.private1.id, aws_subnet.private2.id]
}

output "rds_endpoint" {
  value = aws_db_instance.main.endpoint
}

output "public_instance_ip" {
  value = aws_instance.public_instance.public_ip
}

output "private_instance1_ip" {
  value = aws_instance.private_instance1.private_ip
}

output "private_instance2_ip" {
  value = aws_instance.private_instance2.private_ip
}

