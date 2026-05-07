# Keuangan-Bijak

 
https://keuangan-bijak-blue--Pier211.replit.app

import { useState } from "react";
import { useQueryClient } from "@tanstack/react-query";
import { useListArticles, useDeleteArticle, getListArticlesQueryKey } from "@workspace/api-client-react";
import { Card, CardContent } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { BookOpen, X, Trash2, ChevronRight } from "lucide-react";
import { useToast } from "@/hooks/use-toast";

// Define the Article type based on expected fields
interface Article {
  id: number;
  category: string;
  title: string;
  author: string;
  createdAt: string;
  summary: string;
  content: string;
}

export default function Artikel() {
  const { toast } = useToast();
  const queryClient = useQueryClient();
  const [selected, setSelected] = useState<number | null>(null);

  const { data: articles = [], isLoading } = useListArticles({
    query: { queryKey: getListArticlesQueryKey() },
  }) as { data: Article[]; isLoading: boolean };
  const { mutate: deleteArticle } = useDeleteArticle();

  function handleDelete(id: number, e: React.MouseEvent) {
    e.stopPropagation();
    deleteArticle({ id }, {
      onSuccess: () => {
        toast({ title: "Artikel dihapus" });
        queryClient.invalidateQueries({ queryKey: getListArticlesQueryKey() });
        if (selected === id) setSelected(null);
      },
    });
  }

  const selectedArticle = articles.find((a: Article) => a.id === selected);

  return (
    <div className="py-6 space-y-6">
      <div>
        <h1 className="text-2xl font-bold">Artikel Literasi Keuangan</h1>
        <p className="text-muted-foreground text-sm">Edukasi finansial untuk masa depan yang lebih baik</p>
      </div>

      {selectedArticle && (
        <div className="fixed inset-0 z-50 flex items-end sm:items-center justify-center p-4 bg-black/50 backdrop-blur-sm" onClick={() => setSelected(null)}>
          <div className="bg-card rounded-2xl shadow-2xl w-full max-w-2xl max-h-[85vh] overflow-y-auto" onClick={(e) => e.stopPropagation()}>
            <div className="sticky top-0 bg-card border-b border-border px-6 py-4 flex items-start gap-3">
              <div className="flex-1">
                <Badge className="mb-2">{selectedArticle.category}</Badge>
                <h2 className="text-xl font-bold">{selectedArticle.title}</h2>
                <p className="text-sm text-muted-foreground mt-1">
                  Oleh {selectedArticle.author} &middot; {new Date(selectedArticle.createdAt).toLocaleDateString("id-ID", { day: "numeric", month: "long", year: "numeric" })}
                </p>
              </div>
              <button onClick={() => setSelected(null)} className="p-2 hover:bg-muted rounded-lg transition-colors">
                <X className="w-5 h-5" />
              </button>
            </div>
            <div className="px-6 py-4">
              <p className="text-muted-foreground italic border-l-4 border-primary pl-4 mb-4">{selectedArticle.summary}</p>
              <div className="prose prose-sm max-w-none text-foreground whitespace-pre-wrap">{selectedArticle.content}</div>
            </div>
          </div>
        </div>
      )}

      {isLoading ? (
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
          {[1, 2, 3, 4, 5, 6].map((i) => <div key={i} className="h-48 rounded-xl bg-muted animate-pulse" />)}
        </div>
      ) : articles.length === 0 ? (
        <div className="text-center py-16 text-muted-foreground">
          <BookOpen className="w-16 h-16 mx-auto mb-4 opacity-30" />
          <p className="font-medium text-lg">Belum ada artikel</p>
        </div>
      ) : (
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
          {articles.map((article: Article) => (
            <Card key={article.id} className="cursor-pointer hover:shadow-md hover:border-primary/30 transition-all duration-200 group overflow-hidden" onClick={() => setSelected(article.id)}>
              <div className="h-1.5 bg-gradient-to-r from-blue-500 to-blue-400" />
              <CardContent className="pt-4 pb-4">
                <div className="flex items-start justify-between gap-2 mb-2">
                  <Badge variant="outline" className="text-xs shrink-0">{article.category}</Badge>
                  <button onClick={(e) => handleDelete(article.id, e)} className="text-muted-foreground hover:text-red-500 opacity-0 group-hover:opacity-100 transition-all p-1 rounded shrink-0">
                    <Trash2 className="w-3.5 h-3.5" />
                  </button>
                </div>
                <h3 className="font-semibold text-sm leading-snug mb-2 line-clamp-2">{article.title}</h3>
                <p className="text-xs text-muted-foreground line-clamp-3 mb-3">{article.summary}</p>
                <div className="flex items-center justify-between">
                  <div>
                    <p className="text-xs font-medium text-foreground">{article.author}</p>
                    <p className="text-xs text-muted-foreground">{new Date(article.createdAt).toLocaleDateString("id-ID")}</p>
                  </div>
                  <ChevronRight className="w-4 h-4 text-primary group-hover:translate-x-1 transition-transform" />
                </div>
              </CardContent>
            </Card>
          ))}
        </div>
      )}
    </div>
  );
}
