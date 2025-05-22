This is a [Next.js](https://nextjs.org) project bootstrapped with [`create-next-app`](https://nextjs.org/docs/app/api-reference/cli/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `app/page.js`. The page auto-updates as you edit the file.

This project uses [`next/font`](https://nextjs.org/docs/app/building-your-application/optimizing/fonts) to automatically optimize and load [Geist](https://vercel.com/font), a new font family for Vercel.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/app/building-your-application/deploying) for more details.



"use client";

import { useEffect, useState } from "react";
import axios from "axios";
import { Pagination, Modal, Card, Skeleton } from "antd";
import Image from "next/image";
import { ToastContainer, toast } from "react-toastify";
import styles from "./Estudante.module.css";

const HEADERS = { "x-api-key": process.env.NEXT_PUBLIC_API_KEY };

export default function Estudantes() {
  const [data, setData] = useState({
    estudantes: [],
    loading: true,
    current: 1,
    pageSize: 5,
  });

  const [modalInfo, setModalInfo] = useState({
    visible: false,
    estudante: null,
    avaliacao: null,
    loading: false,
  });

  useEffect(() => {
    const fetchEstudantes = async () => {
      try {
        const { data: estudantes } = await axios.get(
          `${process.env.NEXT_PUBLIC_API_URL}/estudante`,
          { headers: HEADERS }
        );
        setData({ estudantes, loading: false, current: 1, pageSize: 5 });
      } catch {
        toast.error("Erro ao carregar estudantes");
        setData((d) => ({ ...d, loading: false }));
      }
    };

    fetchEstudantes();
  }, []);

  const openModal = async (estudante) => {
    setModalInfo({ visible: true, estudante, avaliacao: null, loading: true });

    try {
      const { data: avaliacao } = await axios.get(
        `${process.env.NEXT_PUBLIC_API_URL}/avaliacao/${estudante.id}`,
        { headers: HEADERS }
      );
      setModalInfo((m) => ({ ...m, avaliacao, loading: false }));
    } catch {
      toast.error("Erro ao carregar avaliação.");
      setModalInfo((m) => ({ ...m, loading: false }));
    }
  };

  const paginatedEstudantes = () => {
    const start = (data.current - 1) * data.pageSize;
    return data.estudantes.slice(start, start + data.pageSize);
  };

  return (
    <div>
      <h1>Lista de Estudantes</h1>
      <Pagination
        current={data.current}
        pageSize={data.pageSize}
        total={data.estudantes.length}
        onChange={(page, size) =>
          setData((d) => ({ ...d, current: page, pageSize: size }))
        }
        showSizeChanger
        pageSizeOptions={["5", "10", "100"]}
      />

      {data.loading ? (
        <Image
          src="/images/loading.gif"
          width={300}
          height={200}
          alt="Loading"
        />
      ) : (
        <div className={styles.cardsContainer}>
          {paginatedEstudantes().map((estudante) => (
            <Card
              key={estudante.id}
              className={styles.card}
              hoverable
              onClick={() => openModal(estudante)}
              cover={
                <Image
                  alt={estudante.name}
                  src={estudante.photo || "/images/220.svg"}
                  width={220}
                  height={220}
                />
              }
            >
              <Card.Meta
                title={estudante.name}
                description={estudante.numero_registro}
              />
            </Card>
          ))}
        </div>
      )}

      <Modal
        title={`Avaliação de ${modalInfo.estudante?.name}`}
        open={modalInfo.visible}
        onCancel={() =>
          setModalInfo({
            visible: false,
            estudante: null,
            avaliacao: null,
            loading: false,
          })
        }
        onOk={() =>
          setModalInfo({
            visible: false,
            estudante: null,
            avaliacao: null,
            loading: false,
          })
        }
        width={600}
      >
        {modalInfo.loading ? (
          <Skeleton active />
        ) : modalInfo.avaliacao ? (
          <div className={styles.avaliacaoInfo}>
            <p>
              <span className={styles.label}>Nota:</span>{" "}
              {modalInfo.avaliacao.nota}
            </p>
            <p>
              <span className={styles.label}>Professor:</span>{" "}
              {modalInfo.avaliacao.professor}
            </p>
          </div>
        ) : (
          <p style={{ textAlign: "center" }}>Avaliação não encontrada.</p>
        )}
      </Modal>
      <ToastContainer position="top-right" autoClose={4500} />
    </div>
  );
}
